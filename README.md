

import javafx.animation.AnimationTimer;
import javafx.application.Application;
import javafx.scene.Group;
import javafx.scene.Scene;
import javafx.scene.canvas.Canvas;
import javafx.scene.canvas.GraphicsContext;
import javafx.scene.paint.Color;
import javafx.stage.Stage;

import java.util.Arrays;
import java.util.Random;

import static java.lang.Math.round;
import static java.lang.Math.sqrt;
import static java.lang.System.*;

/*
 *  Program to simulate segregation.
 *  See : http://nifty.stanford.edu/2014/mccown-schelling-model-segregation/
 *
 * NOTE:
 * - JavaFX first calls method init() and then method start() far below.
 * - To test uncomment call to test() first in init() method!
 *
 */
// Extends Application because of JavaFX (just accept for now)
public class Neighbours extends Application {

    // Enumeration type for the Actors
    enum Actor {
        BLUE, RED, NONE   // NONE used for empty locations
    }

    // Enumeration type for the state of an Actor
    enum State {
        UNSATISFIED,
        SATISFIED,
        NA     // Not applicable (NA), used for NONEs
    }

    final static Random rand = new Random();
    // Below is the *only* accepted instance variable (i.e. variables outside any method)
    // This variable may *only* be used in methods init() and updateWorld()
    Actor[][] world;              // The world is a square matrix of Actors

    // This is the method called by the timer to update the world
    // (i.e move unsatisfied) approx each 1/60 sec.
    void updateWorld() {
        // % of surrounding neighbours that are like me
        final double threshold = 0.7;
        // TODO
        world = nextStateWorld(world, threshold);
    }

    // This method initializes the world variable with a random distribution of Actors
    // Method automatically called by JavaFX runtime (before graphics appear)
    // Don't care about "@Override" and "public" (just accept for now)
    @Override
    public void init() {
        //test();    // <---------------- Uncomment to TEST!

        // %-distribution of RED, BLUE and NONE
        double[] dist = {0.25, 0.25, 0.50};
        // Number of locations (places) in world (square)
        int nLocations = 90000;

        // TODO

        // Should be last

        world = setActors(0.25, 0.25, nLocations);
        fixScreenSize(nLocations);
        world = randWorld(world);


    }


    // ------- Methods ------------------

    // TODO write the methods here, implement/test bottom up

    Actor[][] setActors(double percRed, double percBlue, int location) {
        Actor[][] worldColor = new Actor[getSize(location)][getSize(location)];
        int index = 0;
        int nLocations = worldColor.length * worldColor.length;
        for (int row = 0; row < worldColor.length; row++) {
            for (int column = 0; column < worldColor.length; column++) {
                if (index < percBlue * nLocations) {
                    worldColor[row][column] = Actor.BLUE;
                    index++;
                } else if (index >= percBlue * nLocations && index < (percRed + percBlue) * nLocations) {
                    worldColor[row][column] = Actor.RED;
                    index++;
                } else {
                    worldColor[row][column] = Actor.NONE;
                    index++;
                }


            }
        }
        return worldColor;
    }

    int getSize(int nLocations) {
        return (int) Math.round(sqrt(nLocations));
    }

    Actor[][] randWorld(Actor[][] worldColor) {
        Actor[][] temp = new Actor[1][1];
        for (int row = 0; row < worldColor.length; row++) {
            for (int column = 0; column < worldColor.length; column++) {
                int randRow = Randomize();
                int randCol = Randomize();
                temp[0][0] = worldColor[row][column];
                worldColor[row][column] = worldColor[randRow][randCol];
                worldColor[randRow][randCol] = temp[0][0];

            }
        }
        return worldColor;
    }


    int Randomize() {
        int randNum = rand.nextInt(world.length - 1);
        return randNum;
    }


    boolean isValidLocation(Actor[][] world, int row, int col) {
        int size = world.length;
        return 0 <= row && row < size && 0 <= col && col < size;
    }

    int getSatisfaction(Actor[][] world, int row, int col) {          //hårig kod
        int count = 0;
        if (world[row][col] == Actor.RED) {
            for (int r = row - 1; r <= row + 1; r++) {
                for (int c = col - 1; c <= col + 1; c++) {
                    if (isValidLocation(world, r, c) && !(r == row && c == col)) {
                        if (world[r][c] == Actor.RED) {
                            count++;
                        }
                    }
                }
            }

        } else if (world[row][col] == Actor.BLUE) {
            for (int r = row - 1; r <= row + 1; r++) {
                for (int c = col - 1; c <= col + 1; c++) {
                    if (isValidLocation(world, r, c) && !(r == row && c == col)) {
                        if (world[r][c] == Actor.BLUE ) {                               //|| world[r][c] == Actor.NONE
                            count++;
                        }
                    }
                }
            }
        }
        return count;
    }


    int isOtherColor(Actor[][] world, int row, int col) {          //hårig kod
        int count = 0;
        if (world[row][col] == Actor.RED) {
            for (int r = row - 1; r <= row + 1; r++) {
                for (int c = col - 1; c <= col + 1; c++) {
                    if (isValidLocation(world, r, c) && !(r == row && c == col)) {
                        if (world[r][c] == Actor.BLUE) {
                            count++;
                        }
                    }
                }
            }

        } else if (world[row][col] == Actor.BLUE) {
            for (int r = row - 1; r <= row + 1; r++) {
                for (int c = col - 1; c <= col + 1; c++) {
                    if (isValidLocation(world, r, c) && !(r == row && c == col)) {
                        if (world[r][c] == Actor.RED) {
                            count++;
                        }
                    }
                }
            }
        }
        return count;
    }

   /* boolean isActorNone(Actor[][] world, int row, int col) {
        return world[row][col] == Actor.NONE;
    }*/





    Actor[][] nextStateWorld(Actor[][] world, double threshold){
        int size = world.length;
        Actor[][] newWorld = new Actor[size][size];
        for (int r = 0; r < size; r++){
            for (int c = 0; c < size; c++){
                newWorld[r][c] = Actor.NONE;
            }
        }

        //newWorld = world;
        //Actor[][] newWorld = new Actor[size][size];
        for (int row = 0; row < size; row++) {
            for (int column = 0; column < size; column++) {
                State newState = unSatisfied(world, row, column, threshold);
                if (newState == State.SATISFIED) {
                    newWorld[row][column] = world[row][column];
                }
            }
        }
        for (int row = 0; row < size; row++) {
            for (int column = 0; column < size; column++) {
                State newState = unSatisfied(world, row, column, threshold);
                if (newState == State.UNSATISFIED) {
                    for (; ; ) {
                        int j = rand.nextInt(size);                     //MÅSTE GÖRA NY VÄRLD SOM TAR IN DE SATISFIED OCKSÅ
                        int k = rand.nextInt(size);
                        if (newWorld[j][k] == Actor.NONE) {

                            newWorld[j][k] = world[row][column];

                            break;
                        }


                    }

                }
            }

        }

        return newWorld;
    }










    /*State satisfaction(Actor[][] world, int row, int col, double threshold) {
        double percSat = threshold * aroundYou(world, row ,col);
        if (getSatisfaction(world, row, col) >= percSat || aroundNone(world, row, col)) || ifAllZero(world, row, col) >= percSat {
            return State.SATISFIED;
        } else if (getSatisfaction(world, row, col) < percSat) {
            return State.UNSATISFIED;
        }
        else {
            return State.NA;
        }

    }*/

    State unSatisfied(Actor[][] world, int row, int col, double threshold){
        double percSat = (threshold * (isOtherColor(world, row, col) + getSatisfaction(world, row, col)));              //rundar ner atm
        State s = State.UNSATISFIED;
        if (isOtherColor(world, row, col) > 0){
            if (getSatisfaction(world, row, col) >= percSat){
                s = State.SATISFIED;
            }
        }
        else if (isOtherColor(world, row, col) == 0){
            s = State.SATISFIED;
        } else {
            s = State.NA;
        }
        return s;
    }



    /*int aroundYou(Actor[][] world, int row, int col) {
        int count = 0;
        for (int r = row - 1; r <= row + 1; r++) {
            for (int c = col - 1; c <= col + 1; c++) {
                if (isValidLocation(world, r, c) && !(r == row && c == col)){
                    count++;
                }


            }
        }
        return count;
    }
    int isAllNone(Actor[][] world, int row, int col) {
        int count = 0;
        for (int r = row - 1; r <= row + 1; r++) {
            for (int c = col - 1; c <= col + 1; c++) {
                if (isValidLocation(world, r, c) && !(r == row && c == col) && world[r][c] == Actor.NONE){
                    count++;
                }


            }
        }
        return count;
    }
    boolean aroundNone(Actor[][] world, int row, int col){
        return aroundYou(world, row, col) == isAllNone(world, row, col);
    }
*/
    /*
                                Satisfaction
                                       |
                  ....................................................
                  |                 |                           |
              enum state       getSatisfaction(int)         threshold
                                        |
                                   isValidLocation




*/


    // ------- Testing -------------------------------------

    // Here you run your tests i.e. call your logic methods
    // to see that they really work
    void test() {
        // A small hard coded world for testing
        Actor[][] testWorld = new Actor[][]{
                {Actor.RED, Actor.NONE, Actor.NONE},
                {Actor.NONE, Actor.NONE, Actor.NONE},
                {Actor.RED, Actor.NONE, Actor.BLUE}
        };
        double th = 0.5;   // Simple threshold used for testing
        //int size = testWorld.length;
        //out.println(getSize(9));
        //Actor [][] world2 = setActors(0.25, 0.25, 9);
        //out.println(Arrays.toString(world2[0]));
        //out.println(Arrays.toString(world2[1]));
        //out.println(Arrays.toString(world2[2]));
        //out.println(Arrays.toString(world2[3]));
        // TODO test methods
        //out.println(aroundNone(testWorld, 0, 0));
        //out.println(aroundYou(testWorld, 1, 1));

        exit(0);
    }

    // Helper method for testing (NOTE: reference equality)
    <T> int count(T[] arr, T toFind) {
        int count = 0;

        for (int i = 0; i < arr.length; i++) {
            if (arr[i] == toFind) {
                count++;
            }
        }
        return count;
    }

    // *****   NOTHING to do below this row, it's JavaFX stuff  ******

    double width = 400;   // Size for window
    double height = 400;
    long previousTime = nanoTime();
    final long interval = 95000000;
    double dotSize;
    final double margin = 50;

    void fixScreenSize(int nLocations) {
        // Adjust screen window depending on nLocations
        dotSize = (width - 2 * margin) / sqrt(nLocations);
        if (dotSize < 1) {
            dotSize = 2;
        }
    }

    @Override
    public void start(Stage primaryStage) throws Exception {

        // Build a scene graph
        Group root = new Group();
        Canvas canvas = new Canvas(width, height);
        root.getChildren().addAll(canvas);
        GraphicsContext gc = canvas.getGraphicsContext2D();

        // Create a timer
        AnimationTimer timer = new AnimationTimer() {
            // This method called by FX, parameter is the current time
            public void handle(long currentNanoTime) {
                long elapsedNanos = currentNanoTime - previousTime;
                if (elapsedNanos > interval) {
                    updateWorld();
                    renderWorld(gc, world);
                    previousTime = currentNanoTime;
                }
            }
        };

        Scene scene = new Scene(root);
        primaryStage.setScene(scene);
        primaryStage.setTitle("Simulation");
        primaryStage.show();

        timer.start();  // Start simulation
    }


    // Render the state of the world to the screen
    public void renderWorld(GraphicsContext g, Actor[][] world) {
        g.clearRect(0, 0, width, height);
        int size = world.length;
        for (int row = 0; row < size; row++) {
            for (int col = 0; col < size; col++) {
                double x = dotSize * col + margin;
                double y = dotSize * row + margin;

                if (world[row][col] == Actor.RED) {
                    g.setFill(Color.RED);
                } else if (world[row][col] == Actor.BLUE) {
                    g.setFill(Color.BLUE);
                } else {
                    g.setFill(Color.WHITE);
                }
                g.fillOval(x, y, dotSize, dotSize);
            }
        }
    }

    public static void main(String[] args) {
        launch(args);
    }

}
