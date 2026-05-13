---
theme: css/ucf-knights.css
width: "1920"
height: "1080"
share_cop3223c: "true"
site-folder: docs/Lecture Slides
---

# Debugging C in GitHub Codespaces  
## A Step-by-Step Guide to Native Development 
[view slides online](https://teaching.johnaedo.com/lectures/cop3223c/Debugging%20C%20in%20GitHub%20Codespaces)
  
---
## Prerequisites for C Development  

1. **Extension**: Install the **C/C++ by Microsoft** extension.  
  - We already installed this in the previous lesson.
1. **Compiler**: Ensure `gcc` or `clang` is available.  
  - Codespaces usually come with build-essential packages pre-installed.  
1. **Debugger**: `gdb` (GNU Debugger) is required for interactive debugging.  
<br><br>

> 💡 *Check Terminal*: Run `gcc --version` to verify compilation tools are present.  
  
---
## Configuring the Project Structure  
### `.vscode` Configuration Files  
  
To automate C compilation and debugging, create these files in the `.vscode` directory:  
  
**1. `tasks.json` (Build)**  
```json  
{  
 "version": "2.0.0",  
 "tasks": [  
   {  
     "label": "Build C Program",  
     "command": "gcc",  
     "args": ["-g", "main.c", "-o", "main.out"],  
     "type": "shell"  
   }  
 ]  
}  
```  
---

**2. `c_cpp_properties.json` (IntelliSense)**  
```json  

"configurations": [
	{  
	 "compilerPath": "/usr/bin/gcc",  
	 "cStandard": "c11",  
	 "cppStandard": "c++11"  
	}
],
"version": 4  
```  
<br><br>  
> *Tip*: The `gcc -g` flag ensures debug symbols are generated for the debugger.  
  
---
## Setting Up the Debug Session  
### `launch.json` Configuration  
  
This file tells VS Code how to start the debugger.  
  
**Location**: `.vscode/launch.json`  
  
```json  
{  
 "version": "0.2.0",  
 "configurations": [  
   {  
     "name": "C/C++ Debug",  
     "type": "cppdbg",  
     "request": "launch",  
     "MIMode": "gdb",  
     "miDebuggerPath": "/usr/bin/gdb",  
     "cwd": "${workspaceFolder}",  
     "program": "${workspaceFolder}/main.out",  
     "preLaunchTask": "Build C Program"  
   }  
 ]  
}  
```  
  
> **Auto-build**: The `preLaunchTask` compiles your code automatically before attaching the debugger.  
  
---
## Interactive Debugging Workflow  
### 5 Steps to Debug C Code  
  
1. **Set Breakpoints**: Click the gutter next to lines in `main.c`.  
  - Red dot = active breakpoint.  
2. **Start Debugging**: Press `F5` or click "Run and Debug".  
3. **Step Over/Into**: Use `Step Over` (F10) or `Step Into` (F11).  
4. **Inspect Variables**: Hover over pointers, structs, or arrays.  
5. **Call Stack**: View the execution path and return addresses.  
  
> **Pointer Warning**: When debugging pointers, ensure you inspect the address (`&var`) vs the value (`var`).  
  
---
## Git Integration for C Projects  
### Managing Source Control in Codespaces  
  
- **Untracked Files**: New C files (`main.c`, `debug.h`) are marked as `Untracked`.  
- **Staging**: Stage changes via Source Control panel or `git add main.c`.  
- **Commit**:  
 ```bash  
 git commit -m "fix: null pointer in memory allocation"  
 ```
 <br> <br> 
- **Push**:  
 ```bash  
 git push
 ```  
<br><br>

> **Tip**: Codespaces automatically syncs your working tree. If you break your code, you can `git checkout` to revert.  
  
---
# ⚠ Common Pitfalls  
## Troubleshooting C Debugging  
  
1. **No Symbols**: Ensure `-g` flag is used during compilation.  
  - *Fix*: Update `tasks.json` to include `-g`.  
2. **GDB Path**: Sometimes `gdb` isn't linked in `launch.json`.  
  - *Fix*: Verify path with `which gdb`.  
3. **Port Forwarding**: If your C program creates a server socket, forward the port.  
  - *Fix*: Go to `Port Status` and click the "Open in Browser" icon.  
4. **Async Code**: Debug async C programs requires specific configurations.  
  - *Fix*: Use `Thread` debugger options for multi-threaded C apps.  
  
---
## Summary & Next Steps  
### Recap  
  
- **Extension**: Use C/C++ extension by Microsoft.  
- **Build**: Configure `tasks.json` with `gcc -g`.  
- **Debug**: Configure `launch.json` with GDB.  
- **Git**: Commit changes to sync with GitHub.  

---

## Our Challenge
1. Write a C program with a loop.  
2. Set a breakpoint inside the loop.  
3. Debug until you find the off-by-one error.  
4. Commit and push to GitHub.