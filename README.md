# Bengali Calculator Using Java Servlet
Name: Al-Shahariar

ID:IT-24057

## Objective

Develop a browser-based calculator with a Bengali interface using Java Servlet and Apache Tomcat. The calculator accepts Bengali and English digits and performs addition, subtraction, multiplication, and division.

## Technologies Used

| Technology | Version / Purpose |
| --- | --- |
| Java JDK | 25 |
| Apache Tomcat | 10.1.61 |
| Jakarta Servlet API | HTTP request and response handling |
| HTML and CSS | User interface and styling |
| Visual Studio Code | Source code editing and terminal |
| PowerShell | Compilation and server startup |

## Features

- Accepts Bengali and English digits.
- Supports negative numbers and decimal values.
- Displays calculation results in Bengali digits.
- Handles invalid input and division by zero.
- Preserves the selected operation and normalized input after submission.
- Uses `BigDecimal` for arithmetic and `MathContext.DECIMAL64` for division.

## File Locations

Paths below are relative to the Tomcat installation directory.

| Path | Description |
| --- | --- |
| `hello-src/CalculatorServlet.java` | Servlet source file |
| `lib/servlet-api.jar` | Servlet API supplied with Tomcat |
| `webapps/hello/WEB-INF/classes/CalculatorServlet.class` | Compiled servlet class |

## Source Code

### CalculatorServlet.java

```java
import java.io.IOException;
import java.io.PrintWriter;
import java.math.BigDecimal;
import java.math.MathContext;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

@WebServlet("/calculator")
public class CalculatorServlet extends HttpServlet {

    // Convert Bengali digits to English digits.
    private String normalize(String value) {
        if (value == null) return "";

        StringBuilder result = new StringBuilder();
        for (char c : value.trim().toCharArray()) {
            if (c >= '০' && c <= '৯') {
                result.append((char) ('0' + c - '০'));
            } else {
                result.append(c);
            }
        }
        return result.toString();
    }

    // Format the result using Bengali digits.
    private String banglaNumber(BigDecimal value) {
        String text = value.stripTrailingZeros().toPlainString();
        StringBuilder result = new StringBuilder();

        for (char c : text.toCharArray()) {
            if (c >= '0' && c <= '9') {
                result.append((char) ('০' + c - '0'));
            } else {
                result.append(c);
            }
        }
        return result.toString();
    }

    // Escape user input before inserting it into HTML.
    private String escape(String value) {
        return value.replace("&", "&amp;")
                    .replace("<", "&lt;")
                    .replace(">", "&gt;")
                    .replace("\"", "&quot;")
                    .replace("'", "&#39;");
    }

    private BigDecimal parseNumber(String value) {
        if (value.length() > 30
                || !value.matches("[+-]?[0-9]+(\\.[0-9]+)?")) {
            throw new NumberFormatException();
        }
        return new BigDecimal(value);
    }

    @Override
    protected void doGet(HttpServletRequest request,
                         HttpServletResponse response)
            throws IOException {
        render(response, "", "", "add", "", false);
    }

    @Override
    protected void doPost(HttpServletRequest request,
                          HttpServletResponse response)
            throws IOException {

        request.setCharacterEncoding("UTF-8");

        String first = normalize(request.getParameter("first"));
        String second = normalize(request.getParameter("second"));
        String operation = request.getParameter("operation");

        if (operation == null) operation = "add";

        String message;
        boolean error = false;

        try {
            BigDecimal a = parseNumber(first);
            BigDecimal b = parseNumber(second);
            BigDecimal result;

            switch (operation) {
                case "add":
                    result = a.add(b);
                    break;
                case "subtract":
                    result = a.subtract(b);
                    break;
                case "multiply":
                    result = a.multiply(b);
                    break;
                case "divide":
                    if (b.compareTo(BigDecimal.ZERO) == 0) {
                        throw new ArithmeticException();
                    }
                    result = a.divide(b, MathContext.DECIMAL64);
                    break;
                default:
                    throw new IllegalArgumentException();
            }

            message = "ফলাফল: " + banglaNumber(result);

        } catch (NumberFormatException e) {
            message = "সঠিক সংখ্যা লিখুন। যেমন: ১২, -৫ বা ৩.৫।"
                    + " প্রতিটি সংখ্যা সর্বোচ্চ ৩০ অক্ষরের হতে পারবে।";
            error = true;
        } catch (ArithmeticException e) {
            message = "শূন্য দিয়ে ভাগ করা যায় না।";
            error = true;
        } catch (IllegalArgumentException e) {
            message = "সঠিক গাণিতিক কাজ নির্বাচন করুন।";
            error = true;
        }

        render(response, first, second, operation, message, error);
    }

    private void render(HttpServletResponse response,
                        String first, String second,
                        String operation, String message,
                        boolean error) throws IOException {

        response.setContentType("text/html;charset=UTF-8");
        PrintWriter out = response.getWriter();

        out.println("""
            <!DOCTYPE html>
            <html lang="bn">
            <head>
                <meta charset="UTF-8">
                <meta name="viewport"
                      content="width=device-width, initial-scale=1">
                <title>বাংলা ক্যালকুলেটর</title>
                <style>
                    * { box-sizing: border-box; }
                    body {
                        margin: 0;
                        min-height: 100vh;
                        display: grid;
                        place-items: center;
                        padding: 24px;
                        background: #eef5f2;
                        font-family: system-ui, sans-serif;
                        color: #18382b;
                    }
                    main {
                        width: 100%;
                        max-width: 460px;
                        background: white;
                        padding: 32px;
                        border-radius: 20px;
                        box-shadow: 0 12px 40px #18382b15;
                    }
                    h1 { margin: 0 0 8px; }
                    p { line-height: 1.6; }
                    label {
                        display: block;
                        margin: 18px 0 8px;
                        font-weight: 600;
                    }
                    input, select, button {
                        width: 100%;
                        padding: 12px;
                        font: inherit;
                        border-radius: 10px;
                        border: 1px solid #adc6b8;
                    }
                    button {
                        margin-top: 24px;
                        background: #176b49;
                        color: white;
                        border: none;
                        cursor: pointer;
                    }
                    button:hover { background: #105135; }
                    input:focus, select:focus, button:focus-visible {
                        outline: 3px solid #93d6b8;
                        outline-offset: 2px;
                    }
                    .result {
                        padding: 16px;
                        background: #e5f5ec;
                        border-radius: 10px;
                        overflow-wrap: anywhere;
                    }
                    .error {
                        background: #fff0ed;
                        color: #9c2517;
                    }
                    a { color: #176b49; }
                    small { display: block; margin-top: 16px; }
                </style>
            </head>
            <body>
            <main>
                <h1>বাংলা ক্যালকুলেটর</h1>
                <p>দুটি সংখ্যা লিখে গাণিতিক কাজ নির্বাচন করুন।</p>
                <form method="post" accept-charset="UTF-8">
                    <label for="first">প্রথম সংখ্যা</label>
            """);

        out.println("<input id='first' name='first' type='text'"
                + " maxlength='30' required placeholder='যেমন: ১২.৫'"
                + " value='" + escape(first) + "'>");

        out.println("<label for='operation'>গাণিতিক কাজ</label>");
        out.println("<select id='operation' name='operation'>");

        String[][] options = {
            {"add", "যোগ (+)"},
            {"subtract", "বিয়োগ (−)"},
            {"multiply", "গুণ (×)"},
            {"divide", "ভাগ (÷)"}
        };

        for (String[] option : options) {
            out.println("<option value='" + option[0] + "'"
                    + (option[0].equals(operation) ? " selected" : "")
                    + ">" + option[1] + "</option>");
        }

        out.println("</select>");
        out.println("<label for='second'>দ্বিতীয় সংখ্যা</label>");

        out.println("<input id='second' name='second' type='text'"
                + " maxlength='30' required placeholder='যেমন: ৫'"
                + " value='" + escape(second) + "'>");

        out.println("""
                    <button type="submit">হিসাব করুন</button>
                </form>
            """);

        if (!message.isEmpty()) {
            out.println("<p role='status' class='result"
                    + (error ? " error" : "") + "'>"
                    + escape(message) + "</p>");
        }

        out.println("""
                <p><a href="calculator">নতুন হিসাব</a></p>
                <small>বাংলা ও ইংরেজি অঙ্ক ব্যবহার করা যাবে।
                দশমিকের জন্য ডট (.) লিখুন।
                ভাগের ফল প্রয়োজনে প্রায় ১৬টি উল্লেখযোগ্য অঙ্কে
                রাউন্ড করা হয়।</small>
            </main>
            </body>
            </html>
            """);
    }
}
```

## Compilation and Execution

### 1. Open the Project

Open the Tomcat directory in Visual Studio Code:

```text
D:\Downloads\apache-tomcat-10.1.61
```

Save `CalculatorServlet.java` in the `hello-src` folder using UTF-8 encoding.

### 2. Prepare the Deployment Directory

Open the VS Code PowerShell terminal and run:

```powershell
cd "D:\Downloads\apache-tomcat-10.1.61"
New-Item -ItemType Directory -Force .\webapps\hello\WEB-INF\classes
```

### 3. Compile the Servlet

Stop an existing Tomcat instance before recompiling. Run:

```powershell
javac -encoding UTF-8 -cp ".\lib\servlet-api.jar" -d ".\webapps\hello\WEB-INF\classes" ".\hello-src\CalculatorServlet.java"
```

### 4. Start Tomcat

```powershell
.\bin\catalina.bat run
```

Keep the terminal open while using the application.

### 5. Open the Calculator

Visit [http://localhost:8080/hello/calculator](http://localhost:8080/hello/calculator).

Enter two numbers, select an operation, and click **হিসাব করুন** to calculate the result.

## Working Procedure

1. `doGet()` displays the calculator form.
2. The form submits the two numbers and selected operation through a POST request.
3. `doPost()` reads the request using UTF-8 encoding.
4. `normalize()` converts Bengali digits to English digits.
5. `parseNumber()` validates each input and converts it to `BigDecimal`.
6. The selected arithmetic operation calculates the result.
7. `banglaNumber()` converts the result to Bengali digits.
8. `render()` displays the form with the result or an error message.

## Test Cases and Expected Results

| First Number | Operation | Second Number | Expected Result |
| --- | --- | --- | --- |
| ১২ | Addition | ৮ | ২০ |
| ১০ | Subtraction | ১৫ | -৫ |
| ৩.৫ | Multiplication | ২ | ৭ |
| ২০ | Division | ৪ | ৫ |
| 12 | Addition | ৮ | ২০ |
| ১০ | Division | ০ | Division-by-zero message |
| abc | Addition | ২ | Invalid-number message |

## Input Validation and Precision

Each input supports up to 30 characters. Decimal values use a period, such as `০.৫`. Scientific notation and numbers without an integer part, such as `.5`, are not accepted. Division is rounded to 16 significant digits when necessary. User input is HTML-escaped before being displayed.

## Screenshots

<img width="350" height="416" alt="Screenshot 2026-11-02 085533" src="https://github.com/user-attachments/assets/00183a81-edac-412d-be05-6099000cb9aa" />


<img width="316" height="418" alt="Screenshot 2026-11-02 085616" src="https://github.com/user-attachments/assets/58492544-a5eb-4aba-ac29-b80356e56e0c" />


<img width="305" height="410" alt="Screenshot 2026-11-02 085924" src="https://github.com/user-attachments/assets/432734b7-c743-4cc2-9697-3ce913f8b827" />






## Conclusion

The application demonstrates form processing, request handling, input validation, arithmetic operations, and dynamic HTML responses using Java Servlet. UTF-8 encoding and digit conversion provide a Bengali interface while supporting both Bengali and English numeric input.
