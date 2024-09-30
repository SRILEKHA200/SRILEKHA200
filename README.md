import org.opencv.core.*;
import org.opencv.imgproc.Imgproc;
import org.opencv.objdetect.CascadeClassifier;
import org.opencv.videoio.VideoCapture;

public class DriverSleepDetection {

    static {
        System.loadLibrary(Core.NATIVE_LIBRARY_NAME);
    }

    public static void main(String[] args) {

        // Load Haar Cascade for eye detection
        CascadeClassifier eyeDetector = new CascadeClassifier("path_to_haarcascade/haarcascade_eye.xml");
        VideoCapture capture = new VideoCapture(0); // Open webcam

        if (!capture.isOpened()) {
            System.out.println("Error: Camera not found!");
            return;
        }

        Mat frame = new Mat();
        MatOfRect eyes = new MatOfRect();
        int closedEyeFrames = 0;
        int alertThreshold = 30;  // Threshold for how many frames eyes are closed (can adjust)

        while (true) {
            if (!capture.read(frame)) {
                System.out.println("Error: Unable to capture frame!");
                break;
            }

            // Convert frame to grayscale
            Mat grayFrame = new Mat();
            Imgproc.cvtColor(frame, grayFrame, Imgproc.COLOR_BGR2GRAY);

            // Detect eyes
            eyeDetector.detectMultiScale(grayFrame, eyes);

            // If no eyes are detected, increase the closed-eye frame count
            if (eyes.toArray().length == 0) {
                closedEyeFrames++;
            } else {
                closedEyeFrames = 0;  // Reset if eyes are open
            }

            // If eyes are closed for too long, send alert
            if (closedEyeFrames >= alertThreshold) {
                sendAlert();  // Call alert method
                closedEyeFrames = 0;  // Reset after sending alert
            }

            // Show the frame
            HighGui.imshow("Driver Monitor", frame);

            if (HighGui.waitKey(30) >= 0) {
                break;
            }
        }

        // Release camera and destroy windows
        capture.release();
        HighGui.destroyAllWindows();
    }

    // This method sends an alert (for demo, it's just printing messages)
    public static void sendAlert() {
        System.out.println("Alert: Please move the car left or right!");
        // Optionally: Integrate with a sound system or in-car alert.
    }
}
