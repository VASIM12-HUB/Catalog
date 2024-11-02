import org.json.JSONObject;
import org.json.JSONTokener;

import java.io.FileInputStream;
import java.io.IOException;
import java.math.BigInteger;
import java.util.ArrayList;
import java.util.List;

class Point {
    int x;
    BigInteger y;

    Point(int x, BigInteger y) {
        this.x = x;
        this.y = y;
    }
}

public class SecretPolynomial {

    // Decodes the y-value from a given base to BigInteger for precision
    private static BigInteger decodeBase(String value, int base) {
        return new BigInteger(value, base);
    }

    // Parse input from JSON file
    private static List<Point> parseInput(String filePath) throws IOException {
        List<Point> points = new ArrayList<>();
        int k;

        try (FileInputStream fis = new FileInputStream(filePath)) {
            JSONTokener tokener = new JSONTokener(fis);
            JSONObject jsonData = new JSONObject(tokener);

            k = jsonData.getJSONObject("keys").getInt("k");

            for (String key : jsonData.keySet()) {
                if (!key.equals("keys")) {
                    int x = Integer.parseInt(key);
                    JSONObject rootData = jsonData.getJSONObject(key);
                    int base = rootData.getInt("base");
                    String value = rootData.getString("value");

                    BigInteger y = decodeBase(value, base);
                    points.add(new Point(x, y));
                }
            }
        }

        // Sort points by x and limit to k points if needed
        points.sort((p1, p2) -> Integer.compare(p1.x, p2.x));
        return points.subList(0, k);
    }

    // Perform Lagrange Interpolation to find the constant term
    private static BigInteger lagrangeInterpolationConstant(List<Point> points) {
        BigInteger constantTerm = BigInteger.ZERO;
        int n = points.size();

        for (int i = 0; i < n; i++) {
            BigInteger term = points.get(i).y;

            for (int j = 0; j < n; j++) {
                if (i != j) {
                    int xi = points.get(i).x;
                    int xj = points.get(j).x;

                    // term *= -xj / (xi - xj)
                    term = term.multiply(BigInteger.valueOf(-xj))
                            .divide(BigInteger.valueOf(xi - xj));
                }
            }
            constantTerm = constantTerm.add(term);
        }

        return constantTerm;
    }

    public static void main(String[] args) {
        try {
            String filePath = "testcase.json"; // Path to JSON file
            List<Point> points = parseInput(filePath);
            BigInteger secretConstant = lagrangeInterpolationConstant(points);

            System.out.println("Secret Constant Term (c): " + secretConstant);
        } catch (IOException e) {
            System.out.println("Error reading the file: " + e.getMessage());
        } catch (Exception e) {
            System.out.println("An error occurred: " + e.getMessage());
        }
    }
}
