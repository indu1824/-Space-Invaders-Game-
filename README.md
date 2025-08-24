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

class Alien {
    int row, col;
    int hitsRequired;
    boolean isAlive;
    char symbol;

    Alien(int row, int col, int hitsRequired, char symbol) {
        this.row = row;
        this.col = col;
        this.hitsRequired = hitsRequired;
        this.isAlive = true;
        this.symbol = symbol;
    }
}

class Missile {
    int row, col;
    String type; // "normal", "spread", "laser"

    Missile(int row, int col, String type) {
        this.row = row;
        this.col = col;
        this.type = type;
    }
}

class PowerUp {
    int row, col;
    char type; // 'L'=life, 'B'=bomb, 'S'=score
    boolean active;

    PowerUp(int row, int col, char type) {
        this.row = row;
        this.col = col;
        this.type = type;
        this.active = true;
    }
}

public class Main {
    static Scanner sc = new Scanner(System.in);
    static final int WIDTH = 25;
    static final int HEIGHT = 15;
    static char[][] screen = new char[HEIGHT][WIDTH];
    static Player player;
    static List<Alien> aliens = new ArrayList<>();
    static List<Missile> missiles = new ArrayList<>();
    static List<PowerUp> powerUps = new ArrayList<>();
    static Random rand = new Random();
    static int playerPos = WIDTH/2;
    static Map<String, Integer> highScores = new HashMap<>();
    static final String HIGH_SCORE_FILE = "highscores.txt";
    static int comboCount = 0;

    public static void main(String[] args) throws InterruptedException {
        System.out.print("Enter your name: ");
        String name = sc.nextLine();
        player = new Player(name);
        loadHighScores();

        int level = 1;
        boolean gameOver = false;

        while (!gameOver) {
            initAliens(level);
            System.out.println("\n=== Level " + level + " ===");

            while (!aliens.isEmpty() && player.lives > 0) {
                clearScreen();
                drawScreen();
                showStats();
                System.out.print("Move (a:left, d:right, w:shoot, s:spread, l:laser): ");
                String input = sc.nextLine();
                handleInput(input);
                moveMissiles();
                moveAliens(level);
                movePowerUps();
                checkCollisions();
                checkPowerUps();
                spawnRandomPowerUp();
                Thread.sleep(Math.max(100, 500 - level*30));
            }

            if (player.lives <=0) {
                gameOver = true;
                System.out.println("\nGame Over! Final Score: " + player.score);
            } else {
                System.out.println("Level " + level + " Complete!");
                level++;
            }
        }

        // Save high scores
        if (player.score > highScores.getOrDefault(player.name,0))
            highScores.put(player.name,player.score);
        saveHighScores();
        showLeaderboard();
    }

    static void initAliens(int level){
        aliens.clear();
        int rows = Math.min(3, level+1);
        for(int r=0;r<rows;r++){
            for(int c=0;c<WIDTH;c+=3){
                char symbol='A';
                int hits=1;
                if(rand.nextInt(100)<20){symbol='S';hits=2;} // Shielded alien
                aliens.add(new Alien(r,c,hits,symbol));
            }
        }
        // Boss every 5 levels
        if(level%5==0){
            aliens.add(new Alien(0, WIDTH/2,5,'B'));
        }
    }

    static void clearScreen(){
        for(int i=0;i<HEIGHT;i++)
            Arrays.fill(screen[i],' ');

        for(Alien a: aliens)
            if(a.isAlive) screen[a.row][a.col]=a.symbol;

        for(Missile m: missiles)
            if(m.row>=0 && m.row<HEIGHT) screen[m.row][m.col]='|';

        for(PowerUp p: powerUps)
            if(p.active && p.row>=0 && p.row<HEIGHT) screen[p.row][p.col]=p.type;

        // stars for background
        for(int i=0;i<5;i++) screen[rand.nextInt(HEIGHT)][rand.nextInt(WIDTH)]='*';

        // player
        screen[HEIGHT-1][playerPos]='W';
    }

    static void drawScreen(){
        for(int i=0;i<HEIGHT;i++){
            for(int j=0;j<WIDTH;j++)
                System.out.print(screen[i][j]);
            System.out.println();
        }
    }

    static void handleInput(String input){
        if(input.equalsIgnoreCase("a") && playerPos>0) playerPos--;
        else if(input.equalsIgnoreCase("d") && playerPos<WIDTH-1) playerPos++;
        else if(input.equalsIgnoreCase("w")) missiles.add(new Missile(HEIGHT-2, playerPos,"normal"));
        else if(input.equalsIgnoreCase("s")) {
            missiles.add(new Missile(HEIGHT-2, Math.max(0,playerPos-1),"spread"));
            missiles.add(new Missile(HEIGHT-2, playerPos,"spread"));
            missiles.add(new Missile(HEIGHT-2, Math.min(WIDTH-1,playerPos+1),"spread"));
        } else if(input.equalsIgnoreCase("l")) missiles.add(new Missile(HEIGHT-2, playerPos,"laser"));
        player.shotsFired++;
    }

    static void moveMissiles(){
        List<Missile> toRemove = new ArrayList<>();
        for(Missile m: missiles){
            m.row--;
            if(m.row<0) toRemove.add(m);
        }
        missiles.removeAll(toRemove);
    }

    static void moveAliens(int level){
        for(Alien a: aliens){
            if(!a.isAlive) continue;
            if(rand.nextBoolean()){
                a.col += rand.nextBoolean()?1:-1;
                if(a.col<0) a.col=0;
                if(a.col>=WIDTH) a.col=WIDTH-1;
            }
            a.row++;
            if(a.row>=HEIGHT-1){a.isAlive=false; player.lives--;}
        }
    }

    static void movePowerUps(){
        for(PowerUp p: powerUps){
            if(!p.active) continue;
            p.row++;
            if(p.row>=HEIGHT) p.active=false;
        }
    }

    static void checkCollisions(){
        List<Missile> missilesToRemove = new ArrayList<>();
        for(Missile m: missiles){
            for(Alien a: aliens){
                if(a.isAlive && m.row==a.row && m.col==a.col){
                    a.hitsRequired--;
                    if(a.hitsRequired<=0){
                        a.isAlive=false;
                        player.score+=10;
                        player.aliensDestroyed++;
                        comboCount++;
                        if(comboCount>player.maxCombo) player.maxCombo=comboCount;
                        showExplosion(a.row,a.col);
                        spawnRandomPowerUpAt(a.row,a.col);
                    }
                    missilesToRemove.add(m);
                    break;
                }
            }
        }
        missiles.removeAll(missilesToRemove);
        if(missilesToRemove.isEmpty()) comboCount=0;
    }

    static void spawnRandomPowerUp(){
        if(rand.nextInt(100)<5){ // small chance
            int col = rand.nextInt(WIDTH);
            powerUps.add(new PowerUp(0,col,'S'));
        }
    }

    static void spawnRandomPowerUpAt(int row,int col){
        if(rand.nextInt(100)<20){
            char[] types={'L','B','S'};
            powerUps.add(new PowerUp(row,col,types[rand.nextInt(3)]));
        }
    }

    static void checkPowerUps(){
        Iterator<PowerUp> it = powerUps.iterator();
        while(it.hasNext()){
            PowerUp p = it.next();
            if(p.active && p.row==HEIGHT-1 && p.col==playerPos){
                switch(p.type){
                    case 'L': player.lives++; break;
                    case 'B': // bomb - destroy nearby aliens
                        for(Alien a: aliens)
                            if(a.isAlive && Math.abs(a.row-(HEIGHT-2))<=1 && Math.abs(a.col-playerPos)<=1){
                                a.isAlive=false; player.score+=10; player.aliensDestroyed++; showExplosion(a.row,a.col);
                            }
                        break;
                    case 'S': player.score+=10; break;
                }
                System.out.println("Power-up collected: "+p.type);
                p.active=false;
            }
        }
    }

    static void showExplosion(int row,int col){
        System.out.println("Boom! Alien destroyed at ("+row+","+col+"). Combo x"+comboCount+"!");
    }

    static void showStats(){
        double accuracy = player.shotsFired==0?0: (player.aliensDestroyed*100.0)/player.shotsFired;
        System.out.println("Accuracy: "+String.format("%.1f",accuracy)+"%  Max Combo: "+player.maxCombo);
    }

    static void loadHighScores(){
        try(Scanner fileScanner=new Scanner(new File(HIGH_SCORE_FILE))){
            while(fileScanner.hasNextLine()){
                String[] parts=fileScanner.nextLine().split(",");
                if(parts.length==2) highScores.put(parts[0],Integer.parseInt(parts[1]));
            }
        }catch(Exception e){}
    }

    static void saveHighScores(){
        try(PrintWriter writer=new PrintWriter(new File(HIGH_SCORE_FILE))){
            for(Map.Entry<String,Integer> entry: highScores.entrySet())
                writer.println(entry.getKey()+","+entry.getValue());
        }catch(Exception e){System.out.println("Error saving high scores.");}
    }

    static void showLeaderboard(){ 
        System.out.println("\n--- High Scores ---");
        highScores.entrySet().stream()
                .sorted(Map.Entry.<String,Integer>comparingByValue().reversed())
                .limit(10)
                .forEach(e->System.out.println(e.getKey()+": "+e.getValue()));
    }
}
