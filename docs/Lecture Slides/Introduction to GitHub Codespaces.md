---
theme: css/ucf-knights.css
width: "1920"
height: "1080"
share_cop3223c: "true"
site-folder: docs/Lecture Slides
---
# Introduction to GitHub Codespaces  
## A Complete C Development Workflow  

---
## Creating a Repository & Initializing a Codespace  
  
- **GitHub Repository (a.k.a. repo)**: The source of truth for your code.  
	- This is where your code lives online.
	- We'll discuss local development later, but for now we'll stick to online coding.

- **Codespace**: A cloud-hosted virtual machine (VM) attached to the repo.  
	- This is where we'll be writing, compiling, running, and debugging code. 

---
1. Create a new repository on GitHub.  
 2. Navigate to the **Code** button.  
 3. Select the **Codespaces** tab.  
 4. Click **New Codespace**.  


> ⚡ *The Environment*: VS Code opens in your browser with a cloud-attached container. 

---
## Installing the C/C++ Extension  
  
- **Default Environment**: Codespaces come with Node.js and basic editing by default. 
	- Do this look like a web dev class??? 
- **Missing Features**: No IntelliSense, debugging, or C-specific tooling by default.  
- **Solution**: Install the Microsoft C/C++ Extension.  
  
---
1. Open the **Extensions** panel (Ctrl+X, E).  
2. Search: `C/C++ Extension Pack`.  
3. Click **Install** on `C/C++ Extension Pack`  
 
> **Why?** This adds syntax highlighting, error detection, and debugging support to your editor.  
  
---
## Disabling Copilot Autocompletion  
  
- **Default Behavior**: GitHub Copilot suggests code inline automatically.  
- **Requirement**: In structured tutorials, we want to see the code we write.

AI suggestions can create confusion, especially during the early stages of learning.

---
## How to Disable  
 1. Open **Settings** (Ctrl+,).  
 2. Search for `Disable AI Features`  
 3. Check the option.

  
> **Impact**: Prevents code from being auto-inserted, allowing you to learn the syntax manually.  
  
---
## Writing a Hello World C Program  
  
**File**: `hello.c`

```c  
 #include <stdio.h>  
  
 int main() {  
     printf("Hello, Codespace!\n");  
     return 0;  
 }  
```

---
**Compile & Run**:  
 ```bash  
 gcc hello.c -o hello  
 ./hello  
 ```  
  
> **Goal**: Confirm the environment is compiling and executing C binaries correctly.  
  
---
## Git Basics & Repo Relations  
The VM Nature of Codespaces  
  
### The Workflow:  
1. **Local Repo vs. Remote**:  
  - **Codespace**: The ephemeral VM running on GitHub's servers.  
  - **GitHub Repo**: The permanent storage of your code.  
1. **Synchronization**:  
  - Changes in the Codespace do **not** auto-push to GitHub.  
  - You must manually manage files.

---
### Commands:  
```bash  
git status            # Check uncommitted changes  
git add .             # Stage all files  
git commit -m "hello world" # Save state  
git push              # Push to GitHub  
```  
---
### Key Concept:  
- **VM Nature**: If you close the Codespace without pushing, changes might exist only in the container memory.  
- **Persistence**: You must `git push` to save your work to the permanent repository.  
  
> **Tip**: The code you see in the Repo (GitHub.com) might lag behind the code you are editing in the Codespace until you commit and push.  
  
---
# Summary  
  
1. **Setup**: Open a Codespace from a Repository.  
2. **Tooling**: Install C/C++ Extension.  
3. **Focus**: Disable AI auto-completion.  
4. **Code**: Write and run C programs.  
5. **Version**: Commit and push changes to GitHub.  
  
> **Final Thought**: Codespaces are powerful, but you must treat them as ephemeral containers requiring explicit version control management.