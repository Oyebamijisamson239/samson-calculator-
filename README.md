import javax.swing.*;
import java.awt.*;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;

public class CalculatorApplication {

    // Declare GUI components
    private JFrame frame;
    private JTextField displayField;

    // Constructor to initialize the calculator GUI
    public CalculatorApplication() {
        frame = new JFrame("Simple Calculator");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        
        // Set the size of the frame
        frame.setSize(400, 500); // Adjust width and height as needed

        frame.setLayout(new BorderLayout());

        // Create the display field
        displayField = new JTextField();
        
        // Set preferred size for the text field
        displayField.setPreferredSize(new Dimension(300, 100)); // Adjust width and height as needed
        
        // Set font size for the text field
        displayField.setFont(new Font("Arial", Font.PLAIN, 60)); // Adjust font size as needed
        
        displayField.setEditable(false);
        displayField.setHorizontalAlignment(JTextField.RIGHT);
        frame.add(displayField, BorderLayout.NORTH);

        // Create panel for buttons
        JPanel buttonPanel = new JPanel(new GridLayout(5, 5)); // Increased rows to accommodate "C" button

        // Define button labels
        String[] buttonLabels = {
            "7", "8", "9", "/",
            "4", "5", "6", "*",
            "1", "2", "3", "-",
            "0", ".", "=", "+",
                "C","DEL"
        };
        

        // Add buttons to the panel
        for (String label : buttonLabels) {
            JButton button = new JButton(label);
            
            // Set font size for the button text
            button.setFont(new Font("Arial", Font.PLAIN, 40)); // Adjust font size as needed
            
            button.addActionListener(new ButtonClickListener());
            buttonPanel.add(button);
        }

        frame.add(buttonPanel, BorderLayout.CENTER);

        frame.setVisible(true);
    }

    // ActionListener implementation to handle button clicks
    private class ButtonClickListener implements ActionListener {
        public void actionPerformed(ActionEvent e) {
            String command = e.getActionCommand();

            // Handle button clicks
            if (command.equals("=")) {
                try {
                    // Evaluate expression and round up result
                    double result =
                            eval(displayField.getText());
                    int roundedResult = (int)
                            Math.round(result);
                    
                    displayField.setText(String.valueOf(roundedResult));

                } catch (Exception ex) {
                    displayField.setText("Error");
                }
            } else if (command.equals("C")) {
                // Clear the display field
                displayField.setText("");
            } else if (command.equals("DEL")) {
                //Delete the last character from the display field
                String currentText = displayField.getText();
                if(!currentText.isEmpty()){
                    
                    displayField.setText(currentText.substring(0,currentText.length()-1));
                }
            }else{
                
                // Append clicked button's label to display
                displayField.setText(displayField.getText() + command);
            }
        }

        // Method to evaluate the expression
        private String evaluateExpression(String expression) {
            return String.valueOf(eval(expression));
        }
    }

    // Method to evaluate arithmetic expression
    private double eval(String expression) {
        return new Object() {
            int pos = -1, ch;

            void nextChar() {
                ch = (++pos < expression.length()) ? expression.charAt(pos) : -1;
            }

            boolean whitespace() {
                while (Character.isWhitespace(ch)) nextChar();
                return false;
            }

            double parse() {
                nextChar();
                double x = parseExpression();
                if (pos < expression.length()) throw new RuntimeException("Unexpected: " + (char) ch);
                return x;
            }

            double parseExpression() {
                double x = parseTerm();
                for (;;) {
                    if (whitespace()) continue;
                    if (ch == '+') {
                        nextChar();
                        x += parseTerm();
                    } else if (ch == '-') {
                        nextChar();
                        x -= parseTerm();
                    } else {
                        return x;
                    }
                }
            }

            double parseTerm() {
                double x = parseFactor();
                for (;;) {
                    if (whitespace()) continue;
                    if (ch == '*') {
                        nextChar();
                        x *= parseFactor();
                    } else if (ch == '/') {
                        nextChar();
                        x /= parseFactor();
                    } else {
                        return x;
                    }
                }
            }

            double parseFactor() {
                if (whitespace()) return 0;
                if (ch == '(') {
                    nextChar();
                    double x = parseExpression();
                    if (ch != ')') throw new RuntimeException("Expected: ')'");
                    nextChar();
                    return x;
                }
                if (ch == '+' || ch == '-') {
                    int sign = (ch == '+') ? 1 : -1;
                    nextChar();
                    double x = parseFactor();
                    return sign * x;
                }
                StringBuilder sb = new StringBuilder();
                while ((ch >= '0' && ch <= '9') || ch == '.') {
                    sb.append((char) ch);
                    nextChar();
                }
                if (sb.length() == 0) throw new RuntimeException("Unexpected: " + (char) ch);
                return Double.parseDouble(sb.toString());
            }
        }.parse();
    }

    // Main method to run the calculator
    public static void main(String[] args) {
        SwingUtilities.invokeLater(new Runnable() {
            public void run() {
                new CalculatorApplication();
            }
        });
    }
}
