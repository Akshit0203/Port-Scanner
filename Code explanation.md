
Main code:
```
import sys
import socket
from datetime import datetime
import threading

# Function to scan a port

def scan_port(target,port):
    try:
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        s.settimeout(1)
        result = s.connect_ex((target,port)) #error indictor - if 0, port is open
        if result == 0:
            print(f"Port {port} is open")
        s.close()
    except socket.error as e:
        print(f"Socket error on port {port}: {e}")
    except Exception as e:
        print(f"Unexpected error on port {port}: {e}")

#Main function - argument validation and target definition
def main():
    if len(sys.argv) == 2:
        target = sys.argv[1]
    else:
        print("Invalid number of arguments.")
        print("Usage: python.exe scanner.py <target>")
        sys.exit(1)

    # Resolve the target hostname to an IP address
    try:
        target_ip = socket.gethostbyname(target)
    except socket.gaierror:
        print(f"Error: Unable to resolve hostname {target}")
        sys.exit(1)

    # Add a pretty banner
    print("-" * 50)
    print(f"Scanning target {target_ip}")
    print(f"Time started: {datetime.now()}")
    print("-" * 50)

    try:
        # Use multithreading to scan ports concurrently
        threads = []
        for port in range(1,65536):
            thread = threading.Thread(target=scan_port, args=(target_ip, port))
            threads.append(thread)
            thread.start()

        # Wait for all threads to complete
        for thread in threads:
            thread.join()

    except KeyboardInterrupt:
        print("\nExiting program.")
        sys.exit(0)

    except socket.error as e:
        print(f"Socket error: {e}")
        sys.exit(1)

    print("\nScan completed!")

if __name__ == "__main__":
    main()
```

`sys` → built-in module that lets you interact with the **Python interpreter**.
- We’ll use it for `sys.argv` (command-line arguments) and `sys.exit()`.

 `import socket`
- Imports the **socket module**, which lets you create **network connections** (TCP/UDP).

`from datetime import datetime`
- `from ... import ...` → import just a specific thing from a module.
- `datetime` module → deals with date and time.
- `datetime` class → represents a specific point in time.
- We'll use `datetime.now()` to print when the scan starts.

`import threading`
- Imports the **threading module**, which allows you to run many functions “at the same time” (concurrently).
- We’ll use it to speed up scanning ports.


```
def scan_port(target,port):
```
- `def` → defines a function.
- `scan_port` → function name.
- `(target, port)` → parameters:
    - `target` = IP address or hostname (string).
    - `port` = port number (integer).
This function will scan **one single port** on the target.

    try:
Starts a try block → we expect some code might raise errors (e.g., network issues).
If something goes wrong, we’ll catch it with except.



```
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
```
s → variable (the socket object).
socket.socket() → creates a new socket.
AF = Address Family
INET = Internet (IPv4)
Means: use IPv4 addresses.
socket.SOCK_STREAM
Type of socket: TCP (stream-based).
So this line creates a TCP IPv4 socket that we’ll use to talk to one port.


```
s.settimeout(1)
```
s.settimeout(1) → sets a timeout of 1 second.
If the connection takes more than 1 second, we stop waiting and move on.
This prevents the scanner from hanging forever on a slow or filtered port.



```
result = s.connect_ex((target,port)) #error indictor - if 0, port is open
```
- `s.connect_ex(...)`:
    - This tries to connect to `(target, port)` **but**:
    - Instead of raising an exception on failure, it returns an **error code**.
    - `0` means **success** → port is open.
    - Non-zero means **failure** → port is closed/filtered/etc.
- `(target, port)` → a **tuple** containing:
    - target IP or hostname
    - port number
- `result = ...` → store the return value in `result`.


```
        if result == 0:
            print(f"Port {port} is open")
```
- `if result == 0:` → check if the connect was successful.
- `print(f"...")` → f-string.

Inside the f-string:
- `"Port {port} is open"` → `{port}` is replaced by the actual port number.
- Example: `Port 80 is open`

```
s.close()
```
Close the socket.
Very important to free system resources.

```
except socket.error as e:
        print(f"Socket error on port {port}: {e}")
```
except socket.error as e:
Catches errors related specifically to the socket module.
e → exception object (contains error message).
print(f"...{e}") → show which port and what error happened.

```
def main():
#Main function - argument validation and target definition
```


```
    if len(sys.argv) == 2:
        target = sys.argv[1]
    else:
        print("Invalid number of arguments.")
        print("Usage: python.exe scanner.py <target>")
        sys.exit(1)
```

## 1️⃣ First: what is `sys.argv`?

Imagine you run this in the terminal:

`python scanner.py 192.168.1.1`

Then:

`sys.argv == ["scanner.py", "192.168.1.1"]`

- `sys.argv` → a **list** of what you typed after `python`
    
- `sys.argv[0]` → `"scanner.py"` (the script name)
    
- `sys.argv[1]` → `"192.168.1.1"` (the target you typed)
    

So:

```
sys.argv[0]  # script name 
sys.argv[1]  # target
```

## 2️⃣ `len(sys.argv)`

`len(sys.argv)` = **how many items** are in that list.

Example:

- If you run: `python scanner.py 192.168.1.1`  
    → `sys.argv = ["scanner.py", "192.168.1.1"]`  
    → `len(sys.argv) == 2`
    
- If you run: `python scanner.py` (no target)  
    → `sys.argv = ["scanner.py"]`  
    → `len(sys.argv) == 1`

## 3️⃣ The `if` line

`if len(sys.argv) == 2:`

Read this like normal English:

> “If the number of arguments is exactly 2…”

2 things =

1. Script name
    
2. Target
    

So this checks:

> “Did the user give **one** target (like an IP or domain)?”

## 4️⃣ Inside the `if` block

    `target = sys.argv[1]`

If there **are** 2 items:

- Take the second one → `sys.argv[1]`
    
- Save it into a variable called `target`
    

Example:

`python scanner.py google.com`

Then:

`sys.argv[1] == "google.com" target = "google.com"`

Now your script knows **what to scan**.

## 5️⃣ The `else` part

If `len(sys.argv) != 2` (not equal to 2), that means:

- User **didn’t** give a target  
    or
    
- Gave **too many** things
    

Then this runs:

```
else:     
print("Invalid number of arguments.")
print("Usage: python.exe scanner.py <target>")     
sys.exit(1)
```

### Line by line:

- `print("Invalid number of arguments.")`  
    → Tell the user: “You didn’t type it correctly.”
    
- `print("Usage: python.exe scanner.py <target>")`  
    → Show them **how** to type it correctly.  
    Example: `python.exe scanner.py 192.168.1.1`
    
- `sys.exit(1)`  
    → Stop the program **right now**.  
    → `1` means “there was an error”.  
    (By convention: `0` = OK, non-zero = problem)



```
    # Resolve the target hostname to an IP address
    try:
        target_ip = socket.gethostbyname(target)
```
- `socket.gethostbyname(target)`:
    
    - If `target` is a hostname (like `google.com`):
        
        - It will resolve DNS and return an IP address (e.g., `142.250.182.78`).
            
    - If `target` is already an IP, it just returns it.
        
- `target_ip = ...` → store the result.



```
    except socket.gaierror:
        print(f"Error: Unable to resolve hostname {target}")
        sys.exit(1)
```
`socket.gaierror` is a **specific type of error** from the **socket module**.  
It happens when **DNS resolution fails** — meaning Python can’t turn a hostname into an IP address.
This error happens if DNS resolution fails (invalid domain).

gai = **Get Address Info**
So **gaierror = get-address-info error**.


```
    # Add a pretty banner
    print("-" * 50)
    print(f"Scanning target {target_ip}")
    print(f"Time started: {datetime.now()}")
    print("-" * 50)
```

```
    try:
        # Use multithreading to scan ports concurrently
        threads = []
```
- `try:` → wrap the whole scanning logic to catch interrupts or errors.
    
- Comment: we’re going to use **threads** to speed up scanning.
    
- `threads = []`
    
    - Create an empty list to store thread objects.

```
for port in range(1,65536):
```
`for port in range(1, 65536):`

- Loops from port `1` up to `65535`.
    
- `range` stops **before** `65536`.
    
- This covers **all valid TCP ports** (1–65535).



```
            thread = threading.Thread(target=scan_port, args=(target_ip, port))
```
A **thread** is like starting a _second worker_ that can run code at the same time as the main program.

Instead of scanning ports **one by one**, threads let us scan **many ports at the same time**, making it much faster.

# 🔍 Break down the code word by word

### `threading.Thread(...)`

- Call the `Thread` class from the `threading` module.
    
- This **creates a new thread object**, but doesn’t start it yet.
    

### `target=scan_port`

- `target` means:
    
    > **What function should this thread run?**
    
- Here, `scan_port` is the name of a function.
    
- So the thread will execute the `scan_port()` function.
    

### `args=(target_ip, port)`

- `args` means **the arguments to send to that function**.
    
- Since `scan_port()` expects two parameters:
    
    `def scan_port(target, port):`
    
- We provide them inside parentheses `( )` as a **tuple**:
    
    - `target_ip` → the IP to scan
        
    - `port` → the specific port number


“Create a new thread that will call the function `scan_port()` and give it the values `target_ip` and `port`.”

```
            threads.append(thread)
```
- Add the newly created thread to the `threads` list.
    
- We track them so we can later `.join()` them (wait for them to finish).

            thread.start()
Because this is inside the `for` loop, you’re creating **a new thread for each port**.

⚠️ In real-world tools, you usually limit the number of threads (e.g., a pool), but for learning this is fine.

```
        # Wait for all threads to complete
        for thread in threads:
            thread.join()
            
```
- `for thread in threads:` → loop over the list of all thread objects.
    
- `thread.join()` → wait for each thread to **finish** before continuing.
    
    - This ensures that **all ports have been scanned** before we print "Scan completed!".


```
    except KeyboardInterrupt:
        print("\nExiting program.")
        sys.exit(0)
```
- `except KeyboardInterrupt:`
    
    - This happens when you press `Ctrl + C` in the terminal.
        
- When pressed:
    
    - Print message.
        
    - Exit gracefully with `sys.exit(0)`.


```
    except socket.error as e:
        print(f"Socket error: {e}")
        sys.exit(1)
```
- Catch any global socket errors in the scan loop.
    
- Print the error and exit with status `1`.


```
if __name__ == "__main__":
    main()
```
## 🧠 First: What is `__name__`?

In every Python file, there is a built-in variable called **`__name__`**.

Python automatically sets `__name__` depending on **how** the file is being run.

|How the file is used|What `__name__` becomes|
|---|---|
|The file is **run directly** (like `python scanner.py`)|`"__main__"`|
|The file is **imported** into another Python file|The file’s name, e.g., `"scanner"`|
## 🧠 Then: What does the `if` statement check?

`if __name__ == "__main__":`

This means:

> **Check if this file is being run directly (not imported).**

If the answer is **yes**, then execute:

`main()`

Which means:

> **Run the main() function**

# 🔍 Why is this useful?

Because sometimes a Python file can be:

### ✔️ Run directly (like a program)

`python scanner.py`

### or ✔️ Imported by another script

`import scanner`

If another script imports it, we **do NOT** want to automatically start scanning ports immediately — that would be chaos 😅

So this code **prevents main() from running unless the script is executed directly**.


# 🎯 Simple Example to Understand

### script1.py

`print("Hello from script1")`

### script2.py

`import script1`

If you run:

`python script2.py`

Output:

`Hello from script1`

Because imported file runs automatically.  
Sometimes we **don’t** want that behavior.

So we modify script1:

```
def hello():
    print("Hello from script1")

if __name__ == "__main__":
    hello()
```

Now running script2 prints **nothing** unless we call `hello()` manually.

## TL;DR (What the whole script does)

1. Reads the target from the command line.
    
2. Resolves the target to an IP.
    
3. Prints a header with time.
    
4. For each port from `1` to `65535`:
    
    - Starts a new thread running `scan_port(target_ip, port)`.
        
    - Each thread:
        
        - Tries to connect to that port.
            
        - If success → prints “Port X is open”.
            
5. Waits for all threads to finish.
    
6. Prints “Scan completed!”.

