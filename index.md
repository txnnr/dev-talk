# Exploring Save File Exploits and Modifications in Pokémon Gold
### Understanding ACE, RCE, and Memory Structures

![photo-here](images/home.jpg)




# What Are We Doing?

- Demonstrating save file manipulation in Pokémon Gold.
- Topics covered:
  - **Arbitrary Code Execution (ACE):** Running custom code.
  - **Remote Code Execution (RCE):** Executing code externally.
  - **Memory Structure:** Understanding offsets and targeted data edits.

**Practical Applications:**
- Changing in-game money.
- Ensuring save file integrity with checksums.
- Automating repetitive modifications.

![Save File Placeholder](images/pokemon-rce.jpg)


# Arbitrary Code Execution (ACE)

- **Definition:** Running custom code by exploiting memory or save file vulnerabilities.
- **How It Works:**
  - ACE exploits unused or mismanaged areas of memory.
  - Redirects the game’s logic to execute unintended code paths.
- **Example in Pokémon Gold:**
  - Modifying specific save file offsets to change in-game data.
  - Hijacking existing game logic.
- **Applications:**
  - Speedrunning (e.g., skipping sections of the game).
  - Debugging and testing game mechanics.

  ##### Memory safety issues are a common cause of arbitrary code execution. Vulnerabilities like buffer overflows, use-after-free, and double-free errors allow attackers to manipulate the application’s memory. These flaws can lead to the execution of malicious code by corrupting memory structures or altering the control flow of programs.

![ACE Example Placeholder](images/ace-img.jpg)


# Tools for Performing Arbitrary Code Execution

### **1. Hex Editors**
- **Examples:**  
  - Hex Fiend (Mac).
  - HxD (Windows).  
- **Usage:**  
  - Manually locate and modify offsets within save files or executables.

---

### **2. Memory Debuggers**
- **Examples:**  
  - Cheat Engine: Analyze live memory states in games.
  - Ghidra: Analyze and modify program execution at runtime.
- **Usage:**  
  - Redirect execution to unintended code paths.
  - Dynamically test memory modifications.

---

### **3. Exploit Development Frameworks**
- **Examples:**  
  - GDB (GNU Debugger): Debug and manipulate binaries.
  - IDA Pro: Disassemble and understand program structure.
- **Usage:**  
  - Reverse engineer and develop custom code execution strategies.

![Tools for ACE Placeholder](images/disassembler-1.png)


# Remote Code Execution (RCE)

- **Definition:** Executing code on a system using external tools or data sources.
- **How It Differs From ACE:**
  - ACE occurs locally within the game or system.
  - RCE involves external inputs like save editors or emulators.
- **Example Scenario:**
  - Using a tool (e.g., PKHeX) to inject new data into a save file.
  - Automating repetitive tasks with scripts.
- **Applications:**
  - Bulk modifications (e.g., editing multiple saves).
  - Custom patches for game enhancements.

![photo-here](images/title-img.jpg)

# Tools for Performing Remote Code Execution

### **1. Save File Editors**
- **Examples:**  
  - PKHeX: Edit Pokémon save files.
  - GameShark/Action Replay: Modify in-game memory via external devices.
- **Usage:**  
  - Directly inject or modify data in save files or memory blocks.

![photo-here](images/pkhex.png)
![photo-here](images/gameshark.jpg)
---

### **2. Emulators**
- **Examples:**  
  - VisualBoyAdvance: Load save files and test modifications.
  - RetroArch: Integrate with tools for debugging and testing.
- **Usage:**  
  - Load modified save files to validate changes.
  - Test custom patches in a controlled environment.

![photo-here](images/retroarch.png)
![photo-here](images/vba.jpg)
---

### **3. Scripting Tools**
- **Examples:**  
  - Python (e.g., save file automation).
  - Bash/PowerShell for bulk file edits.
  - IDA to decompile and edit
- **Usage:**  
  - Automate remote modifications across multiple files.
  - Validate and deploy patches efficiently.

![Tools for RCE Placeholder](images/ida.jpg)

# How Memory Works in Save Files

- **Memory as a Data Grid:**
  - Save files are structured like a grid of memory addresses.
  - Each address (or offset) stores specific pieces of data. 
  
  an Offset is simply indicating the distance (displacement) between the beginning of the object and a given element or point. 

  Eg) in the string `abcdef` you would say the character `d` has an offset of 3 from the beginning of the object `a`

- **Key Components:**
  - **Static Memory:** Fixed data (e.g., player name, initial settings).
  - **Dynamic Memory:** Changing data (e.g., money, items, game state).
- **Example in Pokémon Gold:**
  - Money values stored in offsets:
    - `3182–3184` (primary).
    - `9180–9182` (backup).

![Memory Structure Placeholder](images/memory-2.gif)

![Memory Structure Placeholder](images/offsets.png)
# Endianness and Data Decoding

### **Endianness**
- **Definition:** Refers to the order in which bytes are stored in memory.
- **Little Endian:**
  - Least significant byte (LSB - right most number) stored first.
  - Used in Pokémon Gold's save file.
  - Example: The value `3000` is stored as `B8 0B` in memory. 
- **Big Endian:**
  - Most significant byte (MSB - left most number) stored first.
  - Example: The value `3000` is stored as `0B B8` in memory.

   *3000 in hex = `0xBB8`*

![Endianness Diagram Placeholder](images/endian.jpg)

---

### **Example of LSB vs MSB**
- Decimal Value: `4660`  
  - **Binary Representation:** 
    `0001 0010 0011 0100`  
  - **Little Endian (LSB First):** 
    `34 12`  
    
    - Stored as `0011 0100` (`34`) 
      
      then `0001 0010` (`12`)
  
  - **Big Endian (MSB First):** `12 34`  
    - Stored as `0001 0010` (`12`)
    
      then `0011 0100` (`34`)

---

### **Hex/Binary Decoding**
- How Values are Found (manual way):
  - Scanned save files for offsets storing specific data (e.g., money values).
  - Interpreted hexadecimal values and converted them to decimal.
- Example:
  - **Hexadecimal:** `0B B8` → **Binary:** `00001011 10111000` → **Decimal:** `3000`.


Sometimes you will have to do the above & manually scan for addresses while using your emulator + debugger in order to find where things are saved but now there are tons of lookups and forms that provide RAM mappings of addresses to items such as the following:

- https://www.pokecommunity.com/threads/the-offset-reference-please-contribute.369324/
- https://datacrystal.tcrf.net/wiki/Pok%C3%A9mon_Gold_and_Silver/RAM_map



# Checksums and Validation

### **What is a Checksum?**
- **Definition:** A numerical value calculated from a block of data to ensure integrity.
- **Purpose:**  
  - Detects accidental changes to data.  
  - Used in save files to validate that data hasn’t been corrupted.

---

![Checksum Diagram Placeholder](images/checksum.svg)

![Checksum Diagram Placeholder](images/checksum-process.webp)

---

### **Real-World Example of a Checksum**
- **Scenario:** Sending a file over the internet.
- **Steps:**
  1. Sender calculates a checksum (e.g., sum of all bytes) before sending.
  2. Receiver recalculates the checksum on arrival.
  3. If the checksums match, the data is valid; otherwise, it’s corrupted.

---

### **In Pokémon Gold**
- **Checksum Calculation:**
  - Primary Save Block: `0x2000–0x2D68`.
  - Backup Save Block: `0x4000–0x4D68`.  
- **Modified Offsets:**
  - Primary Checksum: `0x2D69`.
  - Backup Checksum: `0x4D69`.
- **Importance:**
  - Without a valid checksum, the game flags the save file as corrupted.

**These checksums for Pokemon Gold were found using the following tool**:

https://github.com/Chase-san/libspec

http://chase-san.github.io/libspec/index.html


# Key Takeaways

- **ACE and RCE:**  
  - Enable precise, targeted changes to memory for custom functionality.
  - Highlight the importance of securing systems against unintended code execution.

- **Memory Structures and Endianness:**  
  - Understanding how data is stored (e.g., offsets and endianness) is critical for manipulating files and debugging.

- **Data Integrity:**  
  - Checksums provide a real-world mechanism to ensure data consistency and validate file integrity.

- **Relevance Beyond Gaming:**  
  - These concepts apply to web development, data validation, security, & database optimization/design practices.

