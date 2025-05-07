# Test Automation with Playwright and Python: A Step-by-Step Tutorial

This tutorial will guide you through creating a simple Python web application using Flask and then writing automated tests for it using Playwright with Python.

**Prerequisites:**

* Python installed.
* VSCode installed with the Python and Playwright extensions.
* Basic understanding of Python.

**Our Goal:**

1.  **Create a Simple Python Web App:** A "Greeter" app that takes a name and displays a greeting.
2.  **Write Playwright Tests:** Verify the page loads and the greeting functionality works as expected.

---

## Step 1: Set Up Your Project Directory

1.  **Open VSCode.**
2.  **Create a new folder** for your project. For this tutorial, we'll name it `playwright_python_tutorial`.
3.  **Open this folder in VSCode** (`File > Open Folder...`).

---

## Step 2: Create the Simple Python Web Application (Flask App)

We'll use Flask, a lightweight Python web framework.

1.  **Create and Activate a Virtual Environment (Recommended):**
    * Open the terminal in VSCode (`Terminal > New Terminal`).
    * Create a virtual environment:
        ```bash
        python -m venv venv
        ```
    * Activate it:
        * **Windows:**
            ```bash
            .\venv\Scripts\activate
            ```
        * **macOS/Linux:**
            ```bash
            source venv/bin/activate
            ```
        Your terminal prompt should now start with `(venv)`.

2.  **Install Flask:**
    * In the activated terminal, run:
        ```bash
        pip install Flask
        ```

3.  **Create the Flask App File:**
    * Inside your `playwright_python_tutorial` folder, create a new file named `app.py`.
    * Paste the following code into `app.py`:

        ```python
        from flask import Flask, request, render_template_string

        app = Flask(__name__)

        # HTML template for our page
        HTML_TEMPLATE = """
        <!DOCTYPE html>
        <html lang="en">
        <head>
            <meta charset="UTF-8">
            <meta name="viewport" content="width=device-width, initial-scale=1.0">
            <title>Greeter App</title>
            <style>
                body { font-family: sans-serif; margin: 20px; background-color: #f4f4f4; color: #333; }
                .container { background-color: #fff; padding: 20px; border-radius: 8px; box-shadow: 0 0 10px rgba(0,0,0,0.1); }
                input[type="text"] { padding: 10px; margin-right: 10px; border: 1px solid #ddd; border-radius: 4px; }
                input[type="submit"] { padding: 10px 15px; background-color: #007bff; color: white; border: none; border-radius: 4px; cursor: pointer; }
                input[type="submit"]:hover { background-color: #0056b3; }
                h1 { color: #333; }
                p { font-size: 1.2em; }
            </style>
        </head>
        <body>
            <div class="container">
                <h1>Greeter</h1>
                <form method="POST">
                    <label for="name_input">Enter your name:</label>
                    <input type="text" id="name_input" name="name_input" placeholder="e.g., Ada">
                    <input type="submit" value="Greet">
                </form>
                {% if greeting_message %}
                    <p id="greeting_message">{{ greeting_message }}</p>
                {% endif %}
            </div>
        </body>
        </html>
        """

        @app.route('/', methods=['GET', 'POST'])
        def home():
            greeting_message = ""
            if request.method == 'POST':
                name = request.form.get('name_input')
                if name:
                    greeting_message = f"Hello, {name}!"
                else:
                    greeting_message = "Please enter a name."
            return render_template_string(HTML_TEMPLATE, greeting_message=greeting_message)

        if __name__ == '__main__':
            # Running on [http://127.0.0.1:5000/](http://127.0.0.1:5000/)
            app.run(debug=True, port=5000)
        ```

4.  **Run the Flask App:**
    * In your VSCode terminal (with `(venv)` activated), execute:
        ```bash
        python app.py
        ```
    * You'll see output like:
        ```
         * Serving Flask app 'app'
         * Debug mode: on
         * Running on [http://127.0.0.1:5000/](http://127.0.0.1:5000/) (Press CTRL+C to quit)
        ```
    * Open `http://127.0.0.1:5000/` in your web browser to see the app.
    * **Keep this terminal running** so Playwright can test the live app.

---

## Step 3: Set Up Playwright

1.  **Open a New Terminal:**
    * In VSCode, open another terminal instance (or use the `+` icon).
    * Ensure your virtual environment `(venv)` is activated in this new terminal.

2.  **Install Playwright:**
    * In this new terminal, run:
        ```bash
        pip install playwright
        ```

3.  **Install Browser Binaries:**
    * Playwright requires browser binaries (Chromium, Firefox, WebKit). Install them:
        ```bash
        playwright install
        ```
    * This download might take a few minutes.

---

## Step 4: Write Your First Playwright Test

1.  **Create a Test File:**
    * In `playwright_python_tutorial`, create a new folder named `tests`.
    * Inside `tests`, create `test_app.py`.
    * Project structure:
        ```
        playwright_python_tutorial/
        ├── venv/
        ├── app.py
        └── tests/
            └── test_app.py
        ```

2.  **Write Test Code in `tests/test_app.py`:**

    ```python
    import pytest # Playwright uses pytest for its test runner
    from playwright.sync_api import Page, expect

    # Base URL of our locally running Flask app
    BASE_URL = "[http://127.0.0.1:5000/](http://127.0.0.1:5000/)"

    def test_page_loads_successfully(page: Page):
        """Test that the main page loads and has the correct title."""
        # Navigate to the app's home page
        page.goto(BASE_URL)

        # Assert that the page title is "Greeter App"
        expect(page).to_have_title("Greeter App")

        # Assert that the H1 heading is present and has the text "Greeter"
        heading = page.locator("h1")
        expect(heading).to_have_text("Greeter")

    def test_greeting_functionality(page: Page):
        """Test that entering a name and submitting shows a greeting."""
        page.goto(BASE_URL)

        # Locate the name input field by its ID
        name_input = page.locator("#name_input")
        # Locate the submit button by its type attribute
        submit_button = page.locator('input[type="submit"]')

        # Define a test name
        test_name = "World"
        # Fill the input field with the test name
        name_input.fill(test_name)
        # Click the submit button
        submit_button.click()

        # Locate the greeting message paragraph by its ID
        greeting_message = page.locator("#greeting_message")
        # Assert that the greeting message contains the expected text
        expect(greeting_message).to_have_text(f"Hello, {test_name}!")

    def test_empty_name_submission(page: Page):
        """Test that submitting an empty name shows an appropriate message."""
        page.goto(BASE_URL)

        # Locate and click the submit button without entering a name
        submit_button = page.locator('input[type="submit"]')
        submit_button.click()

        # Locate the greeting message element
        greeting_message = page.locator("#greeting_message")
        # Assert that the correct message for an empty submission is shown
        expect(greeting_message).to_have_text("Please enter a name.")

    def test_greeting_updates_with_new_name(page: Page):
        """Test that the greeting message updates if a new name is submitted."""
        page.goto(BASE_URL)

        name_input = page.locator("#name_input")
        submit_button = page.locator('input[type="submit"]')

        # First submission
        name_input.fill("Alice")
        submit_button.click()
        greeting_message = page.locator("#greeting_message") # Re-locate or ensure it's still valid
        expect(greeting_message).to_have_text("Hello, Alice!")

        # Second submission with a different name
        # Playwright's fill() method clears the input before typing by default
        name_input.fill("Bob")
        submit_button.click()
        # The greeting_message locator should still be valid for this simple app
        expect(greeting_message).to_have_text("Hello, Bob!")

    # To run these tests from the command line:
    # 1. Ensure your Flask app (app.py) is running.
    # 2. In a separate terminal (with venv activated), run:
    #    playwright test
    ```

    **Key Playwright Concepts Used:**

    * `import pytest`: Playwright integrates with `pytest` for test discovery and execution.
    * `from playwright.sync_api import Page, expect`:
        * `Page`: Represents a browser tab. It's your primary interface for page interactions.
        * `expect`: Playwright's assertion library for verifying page states.
    * `test_... (page: Page)`: `pytest` identifies functions starting with `test_` as test cases. Playwright injects the `page` fixture.
    * `page.goto(URL)`: Navigates to a URL.
    * `page.locator(selector)`: Finds page elements using selectors (CSS, XPath, text, etc.).
        * `#elementId`: CSS ID selector.
        * `input[type="submit"]`: CSS attribute selector.
    * `expect(element).to_have_text(text)`: Asserts an element contains specific text.
    * `expect(page).to_have_title(title)`: Asserts the page title.
    * `element.fill(text)`: Enters text into an input field.
    * `element.click()`: Simulates a click on an element.

---

## Step 5: Run Your Playwright Tests

1.  **Ensure Flask App is Running:** Your `python app.py` command should still be active in one terminal.
2.  **Run Tests:**
    * In your *second* terminal (with `(venv)` activated and where you installed Playwright), navigate to the root of `playwright_python_tutorial`.
    * Execute:
        ```bash
        playwright test
        ```
        Alternatively, to be explicit about the test directory:
        ```bash
        playwright test tests
        ```

3.  **View Results:**
    * The terminal will show test progress and a summary.
    * **HTML Report:** Playwright generates a detailed HTML report. To view it, run:
        ```bash
        playwright show-report
        ```
        This opens the report in your browser, showing test steps, and (on failure or if configured) screenshots and traces.

---

## Step 6: Exploring Further with the VSCode Playwright Extension

The VSCode Playwright extension offers powerful features:

1.  **Test Explorer:**
    * Open the "Testing" tab (beaker icon) in VSCode.
    * If prompted, "Configure Python Tests", choose `pytest`, and select the `tests` directory.
    * Run tests directly from the Test Explorer.

2.  **Record New & Pick Locator:**
    * **Record New:** Command Palette (`Ctrl+Shift+P` or `Cmd+Shift+P`) -> "Playwright: Record new". Interact with your app (`http://127.0.0.1:5000/`), and Playwright generates Python code for your actions. Great for learning and scaffolding tests.
    * **Pick Locator:** While recording, use this to click an element and get a suggested locator.

3.  **Show Trace Viewer:**
    * Traces provide a detailed, step-by-step visual log of your test execution.
    * Run tests with tracing enabled:
        ```bash
        playwright test --headed --trace on
        ```
        * `--headed`: Shows the browser window during tests (good for debugging).
        * `--trace on`: Enables tracing for all tests.
    * After the run, use `playwright show-report` and click on a test to view its trace.

---

## Next Steps and What You've Learned

**You've learned how to:**

* Set up a basic Flask web application.
* Install and configure Playwright for Python.
* Write Playwright tests using `pytest`.
* Use locators to find elements (`page.locator()`).
* Interact with elements (`fill()`, `click()`).
* Make assertions (`expect()`).
* Run tests and interpret the HTML report.
* Utilize features of the VSCode Playwright extension.

**Areas to explore next:**

* **Advanced Locators:** `page.get_by_role()`, `page.get_by_text()`, `page.get_by_label()`, XPath.
* **More Assertions:** `is_visible()`, `is_enabled()`, `to_have_attribute()`, `to_have_css()`.
* **Waiting Strategies:** Explicit waits vs. Playwright's auto-waits.
* **Configuration:** Using `pytest` fixtures and `conftest.py` for test setup, teardown, and shared configurations (like `BASE_URL`).
* **Page Object Model (POM):** A design pattern for creating maintainable and reusable test code by modeling pages/components as classes.
* **Cross-Browser Testing:** Configure tests to run on Chromium, Firefox, and WebKit (`playwright test --browser all`).
* **CI/CD Integration:** Automate your tests in a continuous integration pipeline (e.g., GitHub Actions, GitLab CI, Jenkins).
