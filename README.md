# Unsteable-Students
<br>

> ### **Table of Content**
>  1. [Source file explanation](#source-file-explanation)
>  1. [Description and purpose of additional structures](#description-and-purpose-of-additional-structures)
>  1. [Game flow description](#game-flow-description)
>  1. [AI logic description](#ai-logic-description)
>      - [Card choice](#card-choice)
>      - [Game strategy](#game-strategy)

<br>

<br>
<br>

## Source file explanation
- **caricaMazzo.c**: Loads `mazzo.txt` creating a circular list of cards. <br>
- **header.h**: Contains all structures/enumerations and subprogram prototypes. <br>
- **main.c**: Manages the game flow. <br>
- **chiusura.c**: Deallocates all the circular lists created. <br>
- **preparazione.c**: Prepares the start of the game by creating the various lists (decks, players). <br>
- **salvataggio.c**: Manages all saving functions, saving to the file specified by the user. <br>
- **statistiche.c**: Manages victory statistics, saving and reading game results from a file. <br>
- **logger.c** : Manages the creation of the `log.txt` file, to keep track of the moves performed <br>
- **ANSI.h** : Macros for text formatting <br>
- **turno.c** : Manages the execution of a human player’s turn <br>
- **turnoIA.c** : Manages the execution of an AI player’s turn <br>
- **liste.c** : Implements functions to manage circular lists <br>
- **funzioni.c**: Implements functions for turn execution <br>
- **effetti.c**: Contains all the functions of the various effects used by the AI and non-AI players <br>
<br>

## Description and purpose of additional structures

### Structure `Statistica` 
typedef struct {   <br>
    char nomePersonaggio[NOME]; // Character name <br>
    int vittorieManuale;        // Number of victories in manual games <br>
    int vittorieIA;             // Number of victories by the AI <br>
} Statistica <br>
<br>

## Game flow description

> 
### **1. Program startup (`main.c`)**<br>
- At program startup, the user **enters the name of the save file**.<br>
- The **existence of a previous save** is checked:<br>
  - If it exists, the user can choose to **load the saved game** or **start a new game**.<br>
  - If it does not exist or a new game is chosen, the program moves to the **preparation** phase.<br>

### **2. Preparation (`preparazione.c`)**<br>
- The **complete deck is loaded from the `mazzo.txt` file**.<br>
- The deck is **randomly shuffled**.<br>
- Cards of type **`MATRICOLA`** are separated from the full deck, creating the **`Aula Studio`** deck.<br>
- The remaining cards form the draw deck (**`mazzoPesca`**).<br>
- From **2 to 4 players** are created, with names provided by the user.<br>
- Each player is dealt **5 cards** from the draw deck and one **`MATRICOLA`** card from the `Aula Studio`.<br>

### **3. Main game loop (`main.c`)**<br>
The main loop continues until a **victory condition** is met:<br>

For each (**generic**) turn:<br>
  - An **automatic game save** is performed.<br>
- The current player performs a **mandatory draw** from the draw deck.<br>
- Any card effects with activation **"INIZIO"** or **"SEMPRE"** are triggered.<br>
- The player can choose whether to **draw another card** or **play a card** from their hand, activating its effects.<br>
- At the end of the turn:<br>
  - If the player has **more than 5 cards in hand**, they must discard cards until the maximum limit of 5 cards is reached.<br>
  - It is checked whether a player has reached the **victory condition** (6 or more students in the classroom without the **`INGEGNERE`** card active).<br>
  - If satisfied, the game ends by updating statistics and announcing the winner.<br>

### **4. Game conclusion (`chiusura.c`)**<br>
- At the end of the match, all resources allocated during the game are freed:<br>
  - **Draw deck**, **Aula studio**, **Discard deck**.<br>
  - Player lists and the cards they have in **hand**, in the **classroom**, and in **bonus/malus**.<br>
- A **confirmation message** of program closure is printed.<br>

<br>

## AI logic description

### **Card choice**<br>
During its turn, the AI automatically selects the most appropriate card to play, following logic based on priorities and the current game situation:<br>

- **High Priority**:<br>
  - If it has a **`LAUREANDO`** type card, this card is played immediately to quickly approach the victory condition.<br>

- **Tactical Priority**:<br>
  - If it detects an **immediate threat** (an opponent with **more than 3 students in the classroom**), the AI attempts to counter it using cards with effects such as:<br>
    - **`ELIMINA`** (targeting an opponent).<br>
    - **`RUBA`** (cards that steal resources from opponents).<br>
    - **`MALUS`** cards that directly penalize opponents.<br>

- **Standard Priority (fallback)**:<br>
  - If there are no evident threats or high-priority cards available, the AI follows a hierarchy in choosing the most useful cards:<br>
    1. **`STUDENTE_SEMPLICE`** cards (increase the number of students in the classroom).<br>
    2. **`BONUS`** cards (persistent positive effects).<br>
    3. **`MAGIA`** cards (temporary special effects).<br>
    4. **`MALUS`** cards (used when no better options are available).<br>

- **If no card is useful**:<br>
  - The AI opts to **automatically draw an additional card** from the draw deck, thus increasing the options available in subsequent turns.<br>

### **Game strategy**<br>
The overall strategy followed by the AI is based on the following key points:<br>

- **Quickly maximize the number of students in the classroom** to reach the victory condition (6 students without the **`INGEGNERE`** card active).<br>
- **Monitor and counter the strongest opponents**, namely those closest to victory.<br>
- **Use effect cards wisely**, saving negative effects such as **`ELIMINA`** or **`RUBA`** to slow down opponents who are ahead.<br>
- **Automatically manage discards** at the end of the turn, always keeping a maximum of **5 cards in hand** and automatically discarding the least useful cards according to the priority: **`MAGIA` → `MALUS` → `BONUS` → `STUDENTE_SEMPLICE` → `LAUREANDO`**.<br>
