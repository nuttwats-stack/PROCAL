import javafx.application.Application;
import javafx.animation.AnimationTimer;
import javafx.scene.Scene;
import javafx.scene.layout.StackPane;
import javafx.scene.canvas.Canvas;
import javafx.scene.canvas.GraphicsContext;
import javafx.scene.paint.Color;
import javafx.stage.Stage;

/**
 * GEG-4 Cooling Tower Basin Simulator (JavaFX Implementation)
 * รวม Logic การคำนวณและ Rendering ไว้ในไฟล์เดียว
 */
public class CoolingTowerSimulator extends Application {

    // Simulation Constants
    private final double BASIN_AREA = 47.39;
    private double level = 88.01;
    private double speedMultiplier = 1.0;
    private long lastTick = System.nanoTime();

    @Override
    public void start(Stage primaryStage) {
        Canvas canvas = new Canvas(800, 600);
        GraphicsContext gc = canvas.getGraphicsContext2D();

        // Simulation Loop
        new AnimationTimer() {
            @Override
            public void handle(long now) {
                double dt = (now - lastTick) / 1_000_000_000.0;
                lastTick = now;
                
                updateDynamics(dt);
                render(gc);
            }
        }.start();

        StackPane root = new StackPane(canvas);
        primaryStage.setTitle("GEG-4 Dynamic Loop Balance Controller");
        primaryStage.setScene(new Scene(root, 800, 600, Color.web("#0B111E")));
        primaryStage.show();
    }

    private void updateDynamics(double dt) {
        // SCADA Logic (จำลองจาก JavaScript)
        double pressure = 6.4;
        double pressScale = Math.sqrt(pressure / 6.4);
        double floatChoke = (level < 90.0) ? 1.0 : (100.0 - level) / 10.0;
        
        double inflow = 5.6 * floatChoke * pressScale;
        double outflow = 4.0 + 2.0; // Blowdown + Thermal Loss
        
        double netFlow = inflow - outflow;
        double volumeDelta = netFlow * (dt * speedMultiplier / 3600.0);
        double levelDelta = (volumeDelta / (BASIN_AREA * 0.250)) * 100.0;
        
        level = Math.max(0, Math.min(110, level + levelDelta));
    }

    private void render(GraphicsContext gc) {
        gc.setFill(Color.web("#0B111E"));
        gc.fillRect(0, 0, 800, 600);

        // วาดระดับน้ำ (Basin Graphic)
        gc.setFill(Color.web("#00b0ff"));
        double h = (level / 100.0) * 200;
        gc.fillRect(300, 400 - h, 200, h);

        // แสดงผลตัวเลข
        gc.setFill(Color.WHITE);
        gc.fillText("BASIN LEVEL: " + String.format("%.2f", level) + "%", 50, 50);
        gc.fillText("STATUS: " + (level > 100 ? "OVERFLOW" : "STABLE"), 50, 70);
    }

    public static void main(String[] args) {
        launch(args);
    }
}
