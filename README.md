# Space Invaders Game 🎮

This is a console-based **Java Space Invaders Game** with scoring system, power-ups, alien/missile mechanics, and leaderboard.

---

## Code Preview (Main.java)

```java
import java.util.*;
import java.io.*;

class Player {
    String name;
    int score;
    int lives;
    int shotsFired;
    int aliensDestroyed;
    int maxCombo;

    Player(String name) {
        this.name = name;
        this.score = 0;
        this.lives = 3;
        this.shotsFired = 0;
        this.aliensDestroyed = 0;
        this.maxCombo = 0;
    }
}

// ... rest of the classes (Alien, Missile, PowerUp, Main) ...


## Screenshots

### Gameplay
![Alt Text](https://github.com/indu1824/-Space-Invaders-Game-/raw/main/Screenshot%202025-08-24%20144528.png)

### Scoreboard
![Alt Text](https://github.com/indu1824/-Space-Invaders-Game-/raw/main/Screenshot%202025-08-24%20144659.png)
![Alt Text](https://github.com/indu1824/-Space-Invaders-Game-/raw/main/Screenshot%202025-08-24%20145038.png)

### Power-ups
![Alt Text](https://github.com/indu1824/-Space-Invaders-Game-/raw/main/Screenshot%202025-08-24%20145526.png)
