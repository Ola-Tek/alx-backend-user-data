
AUTH = Auth()
The end-point should expect two form data fields: "email" and "password". If the user does not exist, the end-point should register it and respond with the following JSON payload:
{"email": "<registered email>", "message": "user created"}
If the user is already registered, catch the exception and return a JSON payload of the form
{"message": "email already registered"}
and return a 400 status code.
Remember that you should only use AUTH in this app. DB is a lower abstraction that is proxied by Auth.
 8. Credentials validation
auth.py contains the following updates:

Implement the Auth.valid_login method. It should expect email and password required arguments and return a boolean.
Try locating the user by email. If it exists, check the password with bcrypt.checkpw. If it matches return True. In any other case, return False.
 9. Generate UUIDs
auth.py contains the following updates:

Implement a _generate_uuid function in the auth module. The function should return a string representation of a new UUID. Use the uuid module.
Note that the method is private to the auth module and should NOT be used outside of it.
 10. Get session ID
auth.py contains the following updates:

Implement the Auth.create_session method. It takes an email string argument and returns the session ID as a string.
The method should find the user corresponding to the email, generate a new UUID and store it in the database as the user’s session_id, then return the session ID.
Remember that only public methods of self._db can be used.
 11. Log in
app.py contains the following updates:

Implement a login function to respond to the POST /sessions route.
The request is expected to contain form data with "email" and a "password" fields.
If the login information is incorrect, use flask.abort to respond with a 401 HTTP status.
Otherwise, create a new session for the user, store it the session ID as a cookie with key "session_id" on the response and return a JSON payload of the form:
{"email": "<user email>", "message": "logged in"}
 12. Find user by session ID
auth.py contains the following updates:

Implement the Auth.get_user_from_session_id method. It takes a single session_id string argument and returns the corresponding User or None.
If the session ID is None or no user is found, return None. Otherwise return the corresponding user.
Remember to only use public methods of self._db.
 13. Destroy session
auth.py contains the following updates:

Implement Auth.destroy_session. The method takes a single user_id integer argument and returns None.
The method updates the corresponding user’s session ID to None.
Remember to only use public methods of self._db.
 14. Log out
app.py contains the following updates:

Implement a logout function to respond to the DELETE /sessions route.
The request is expected to contain the session ID as a cookie with key "session_id".
Find the user with the requested session ID. If the user exists destroy the session and redirect the user to GET /. If the user does not exist, respond with a 403 HTTP status.
 15. User profile
app.py contains the following updates:

Implement a profile function to respond to the GET /profile route.
The request is expected to contain a session_id cookie. Use it to find the user. If the user exist, respond with a 200 HTTP status and the following JSON payload:
{"email": "<user email>"}
If the session ID is invalid or the user does not exist, respond with a 403 HTTP status.
 16. Generate reset password token
auth.py contains the following updates:

Implement the Auth.get_reset_password_token method. It takes an email string argument and returns a string.
Find the user corresponding to the email. If the user does not exist, raise a ValueError exception. If it exists, generate a UUID and update the user’s reset_token database field. Return the token.
 17. Get reset password token
app.py contains the following updates:

Implement a get_reset_password_token function to respond to the POST /reset_password route.
The request is expected to contain form data with the "email" field.
If the email is not registered, respond with a 403 status code. Otherwise, generate a token and respond with a 200 HTTP status and the following JSON payload:
{"email": "<user email>", "reset_token": "<reset token>"}
 18. Update password
auth.py contains the following updates:

Implement the Auth.update_password method. It takes reset_token string argument and a password string argument and returns None.
Use the reset_token to find the corresponding user. If it does not exist, raise a ValueError exception.
Otherwise, hash the password and update the user’s hashed_password field with the new hashed password and the reset_token field to None.
 19. Update password end-point
app.py contains the following updates:

Implement the update_password function in the app module to respond to the PUT /reset_password route.
The request is expected to contain form data with fields "email", "reset_token" and "new_password".
Update the password. If the token is invalid, catch the exception and respond with a 403 HTTP code.
If the token is valid, respond with a 200 HTTP code and the following JSON payload:
{"email": "<user email>", "message": "Password updated"}
 20. End-to-end integration test

Start the Flask app you created in the previous tasks. Open a new terminal window.
Create a new module called main.py. Create one function for each of the following sub tasks. Use the requests module to query your web server for the corresponding end-point. Use assert to validate the response’s expected status code and payload (if any) for each sub task:
register_user(email: str, password: str) -> None.
log_in_wrong_password(email: str, password: str) -> None.
log_in(email: str, password: str) -> str.
profile_unlogged() -> None.
profile_logged(session_id: str) -> None.
log_out(session_id: str) -> None.
reset_password_token(email: str) -> str.
update_password(email: str, reset_token: str, new_password: str) -> None.
Copy the following code at the end of the main module:
EMAIL = "guillaume@holberton.io"
PASSWD = "b4l0u"
NEW_PASSWD = "t4rt1fl3tt3"


if __name__ == "__main__":

    register_user(EMAIL, PASSWD)
    log_in_wrong_password(EMAIL, NEW_PASSWD)
    profile_unlogged()
    session_id = log_in(EMAIL, PASSWD)
    profile_logged(session_id)
    log_out(session_id)
    reset_token = reset_password_token(EMAIL)
    update_password(EMAIL, reset_token, NEW_PASSWD)
    log_in(EMAIL, NEW_PASSWD)
Run python3 main.py. If everything is correct, you should see no output.
RESOURCES
