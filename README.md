# CATALOG
online Assessment 
import org.json.JSONObject;
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Paths;
import java.math.BigInteger;
import java.util.ArrayList;
import java.util.List;

public class ShamirSecretSharing {

    public static void main(String[] args) {
        try {
            // Read JSON file
            String content = new String(Files.readAllBytes(Paths.get("path_to_json_file.json")));
            JSONObject json = new JSONObject(content);

            // Parse n and k
            JSONObject keys = json.getJSONObject("keys");
            int n = keys.getInt("n");
            int k = keys.getInt("k");

            // Parse (x, y) pairs
            List<Point> points = new ArrayList<>();
            for (String key : json.keySet()) {
                if (!key.equals("keys")) {
                    int x = Integer.parseInt(key);
                    JSONObject point = json.getJSONObject(key);
                    int base = point.getInt("base");
                    String value = point.getString("value");

                    BigInteger y = new BigInteger(value, base);
                    points.add(new Point(BigInteger.valueOf(x), y));
                }
            }

            // Calculate constant term 'c' using Lagrange interpolation
            BigInteger secret = calculateConstantTerm(points, k);
            System.out.println("The constant term (secret) is: " + secret);

        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    // Represents a point (x, y)
    static class Point {
        BigInteger x, y;
        Point(BigInteger x, BigInteger y) {
            this.x = x;
            this.y = y;
        }
    }

    // Lagrange interpolation to find the constant term
    public static BigInteger calculateConstantTerm(List<Point> points, int k) {
        BigInteger constantTerm = BigInteger.ZERO;

        // Apply Lagrange interpolation
        for (int i = 0; i < k; i++) {
            BigInteger xi = points.get(i).x;
            BigInteger yi = points.get(i).y;

            BigInteger term = yi;
            for (int j = 0; j < k; j++) {
                if (i != j) {
                    BigInteger xj = points.get(j).x;
                    term = term.multiply(xj).divide(xj.subtract(xi));
                }
            }
            constantTerm = constantTerm.add(term);
        }

        return constantTerm;
    }
}
