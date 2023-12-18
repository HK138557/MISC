# MISC

I am new to GITHUb as a recent programmer, and have been learning using flask to communicate between python and javascript for a website I'm working on.

Currently am on the signup/login page and I have a signup.html page with a form for 3 user inputs: username, password and email address. I have tried to use flask so that for a valid form submission the 3 user inputs are posted from the html based form using POST method to an app.py file, where they are then added to a sqlite3 database provided the email/username is not already in the database. Currently I am just checking to make sure whenever the submit button is clicked and all 3 fields are filled, the entry is added to the database, for ehich I am using writing to a text file. However whenever I try to entr sample usr details which are valid and press submit, I check the text file and it is empty, meaning the database has not been appended to. When i press inspect and chek the console on the html signup page, it shows this error: POST http://127.0.0.1:5500/submit 405 (Method Not Allowed), signupPage line 20. 

Would really appreciate if someone could take time out and help with this a it is quite important.

My code is attached below for app.py and signupPage (html and js)  - 

signupPage.html:

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Signup - Educational Car Racing Game</title>
    <link rel="stylesheet" href="styles.css">
</head>
<body>
    <header>
        <h1>Signup</h1>
        <nav>
            <ul>
                <li><a href="loginPage.html">Already have an account? Login here</a></li>
                <li><a href="homepage.html">Return to Homepage</a></li>
            </ul>
        </nav>
    </header>
    <main>
        <!-- Signup form -->
        <form id="signupForm">
            <input type="text" placeholder="Username" id="username">
            <input type="password" placeholder="Password" id="password">
            <input type="email" placeholder="Email" id="email">
            <button type="submit">Sign Up</button>
        </form>
    </main>
    <footer>
        <!-- Footer content -->
    </footer>
    <script src="signupPage.js"></script>
</body>
</html>




signupPage.js: 

document.getElementById('signupForm').addEventListener('submit', function(e) {
    
    e.preventDefault();

    // Get form data
    const username = document.getElementById('username').value;
    const password = document.getElementById('password').value;
    const email = document.getElementById('email').value;

    // Validate form data (you might want to add more validation)
    if (username && password && email) {
        // Prepare data to send
        const data = {
            username: username,
            password: password,
            email: email
        };

        // Send data to the server using AJAX
        fetch ('/submit', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify(data)
        })
        .then(response => response.json())
        .then(result => {
            // Handle server response (if needed)
            window.alert(result);
        })
        .catch(error => {
            // Handle errors
            window.alert('Error:', error);
        });
    } else {
        window.alert('Please fill in all fields');
    }
});




app.py:

from flask import Flask, request, jsonify
import sqlite3
import hashlib

app = Flask(__name__)

# Create a connection to a SQLite database (or open if it exists)
conn = sqlite3.connect('userdata.db')

# Create a cursor object to execute SQL queries
cursor = conn.cursor()

# Create a table if it doesn't exist already
cursor.execute("CREATE TABLE IF NOT EXISTS users(id INTEGER PRIMARY KEY AUTOINCREMENT,  username TEXT, password TEXT, email TEXT UNIQUE)")

conn.commit()

@app.route('/submit', methods=['GET','POST'])
def submit():
    if request.method == 'POST':
        data = request.get_json()

        # Access data received from JavaScript
        username = data.get('username')
        password = data.get('password')
        email = data.get('email')

        # Hash the password before storing
        hashed_password = hashlib.sha256(password.encode()).hexdigest()

        # Store the username, password, hashed password and email address in text file.
        with open('TEXT.txt', 'a') as file:
                file.write(f"Username: {username}, Password: {password}, Hashed Password: {hashed_password}, Email: {email}\n")

        # Check if the email already exists in the database
        cursor.execute("SELECT * FROM users WHERE email=?", (email,))
        existing_user = cursor.fetchone()

        if existing_user:
            response = {'message': 'Email already exists in the database'}
        else:
            # Insert new user into the database with hashed password
            cursor.execute("INSERT INTO users (username, password, email) VALUES (?, ?, ?)", (username, hashed_password, email))
            conn.commit()
            response = {'message': 'Data received and added to the database'}

        return jsonify(response)

if __name__ == '__main__':
    app.run(debug=True)

# Close the database connection when the app exits
conn.close()








