.. _step13:

**********
JavaScript
**********

.. image:: ./images/AWSServerlessWebApplication-JavaScript.jpg
  :width: 720 px
  :alt: AWS Serverless Web App
  :align: center

The basics of our app are now complete. The next thing to tackle is to fix up our HTML. It is good practise to actually remove the JavaScript code from within the HTML documents and place it in a seperate file. This way the code and content are seperate. What we will do to start is remove the **get_user()** function from ``profile.html`` and call it from our HTML. We will then remove the JavaScript code from ``sign-in.html`` and ``sign-out.html`` and create ``*.js`` files for the JavaScript code in each of these files. When done, all JavaScript will be moved out of the HTML files and into their own files.

Tasks:

- create a js directory and a JavaScript file for our code
- move the **<script>** code from ``sign-in.html`` into a ``sign-in.js`` file
- fix the code up, so that there are no **global** variables, since all code must now be in functions
- call the new function from ``sign-in.html``
- do the same process to ``sign-out.html`` and ``profile.html``

Sign In Code
************

.. tabs::

  .. group-tab:: sign-in.html

    .. code-block:: html
        :linenos:

        <!DOCTYPE html>
        <html lang="en">
          <head>
            <meta charset="utf-8">
            
            <!-- Javascript SDKs-->
            <script src="https://code.jquery.com/jquery-3.4.1.min.js"></script>
            <script src="js/amazon-cognito-auth.min.js"></script>
            <script src="https://sdk.amazonaws.com/js/aws-sdk-2.596.0.min.js"></script> 
            <script src="js/amazon-cognito-identity.min.js"></script>   
            <script src="js/config.js"></script>
            <script type="text/javascript" src="js/sign-in.js"></script>
          </head>
          
          <body>
            <form>
              <h1>Please sign in</h1>

              <input type="text" id="inputUsername"  placeholder="Email address" name="username" required autofocus>
              <input type="password" id="inputPassword"  placeholder="Password" name="password" required>    
              <button type="button" onclick="signInButton();">Sign in</button>
            </form>
            
            <br>
            <div id='logged-in'>
              <p></p>
            </div>
            
            <p><a href="./profile.html">Profile</a></p>
            
            <br>
            <div id='home'>
              <p>
                <a href='./index.html'>Home</a>
              </p>
            </div>
            
          </body>
        </html>

  .. group-tab:: sign-in.js
  
    .. code-block:: javascript
        :linenos:

        // JavaScript File

        function signInButton() {
          // sign-in to AWS Cognito
          
          var data = { 
              UserPoolId : _config.cognito.userPoolId,
            ClientId : _config.cognito.clientId
          };
          var userPool = new AmazonCognitoIdentity.CognitoUserPool(data);
          var cognitoUser = userPool.getCurrentUser();

            var authenticationData = {
            Username : document.getElementById("inputUsername").value,
            Password : document.getElementById("inputPassword").value,
          };

          var authenticationDetails = new AmazonCognitoIdentity.AuthenticationDetails(authenticationData);

          var poolData = {
            UserPoolId : _config.cognito.userPoolId, // Your user pool id here
            ClientId : _config.cognito.clientId, // Your client id here
          };

          var userPool = new AmazonCognitoIdentity.CognitoUserPool(poolData);

          var userData = {
            Username : document.getElementById("inputUsername").value,
            Pool : userPool,
          };

          var cognitoUser = new AmazonCognitoIdentity.CognitoUser(userData);

          cognitoUser.authenticateUser(authenticationDetails, {
            onSuccess: function (result) {
              var accessToken = result.getAccessToken().getJwtToken();
              console.log(result);  
              
              //get user info, to show that you are logged in
                    cognitoUser.getUserAttributes(function(err, result) {
                        if (err) {
                            console.log(err);
                            return;
                        }
                        console.log(result);
                        document.getElementById("logged-in").innerHTML = "You are logged in as: " + result[2].getValue();
                        
                        // now auto redirect to profile page
                        window.location.replace("./profile.html");
                    });
              
            },
            onFailure: function(err) {
              alert(err.message || JSON.stringify(err));
            },
          });
        }

Sign Out Code
************

.. tabs::

  .. group-tab:: sign-out.html

    .. code-block:: html
        :linenos:

        <!doctype html>
        <html lang="en">
          <head>
            <meta charset="utf-8">
            <!--Cognito JavaScript-->
            <script src="js/amazon-cognito-identity.min.js"></script>  
            <script src="js/config.js"></script>
            <script src="js/sign-out.js"></script>
          </head>

          <body>
            <div class="container">
              <div>
                <h1>Sign Out</h1>
                <div id='sign-out'>
                  <p>One moment please ...</p>
                </div>
              </div>
            <div>
              
            <br>
            <div id='home'>
              <p>
                <a href='./index.html'>Home</a>
              </p>
            </div>
          </body>
          <script>
            window.onload = function(){
              const temp_var = signOut();
            }
          </script>
        </html>

  .. group-tab:: sign-out.js
  
    .. code-block:: javascript
        :linenos:

        // JavaScript File

        function signOut() {
          //
          
          return_message = "";
          
          const data = { 
            UserPoolId : _config.cognito.userPoolId,
            ClientId : _config.cognito.clientId
          };
          const userPool = new AmazonCognitoIdentity.CognitoUserPool(data);
          const cognitoUser = userPool.getCurrentUser();

          if (cognitoUser != null) {
            cognitoUser.getSession(function(err, session) {
              if (err) {
                alert(err);
                return;
              }
              console.log('session validity: ' + session.isValid());

              // sign out
              cognitoUser.signOut();
              console.log("Signed-out");
              return_message = "Signed-out";
            });
          } else {
            console.log("Already signed-out")
            return_message = "Already signed-out";
          }
          
          const div_user_info = document.getElementById('sign-out');
          div_user_info.innerHTML = return_message;
        }

Profile Code
************

.. tabs::

  .. group-tab:: profile.html

    .. code-block:: html
        :linenos:

        <!doctype html>
        <html lang="en">
          <head>
            <meta charset="utf-8">
            <!--Cognito JavaScript-->
            <script src="js/amazon-cognito-identity.min.js"></script>  
            <script src="js/config.js"></script>
            <script type="text/javascript" src="js/profile.js"></script>
          </head>

          <body>
            <div class="container">
              <div>
                <h1>Profile</h1>
              </div>
              <div id='profile'>
                <p>One moment please ...</p>
              </div>
            <div>
              
            <br>
            <div id='home'>
              <p>
                <a href='./index.html'>Home</a>
              </p>
            </div>
          </body>
          <script>
            window.onload = function(){
              const temp_var = getUserAttributes();
            }
          </script>
        </html>

  .. group-tab:: profile.js
  
    .. code-block:: javascript
        :linenos:
        
        // JavaScript File
    
        async function getUser(email_address) {
            // get the user info from API Gate
            
            const api_url = 'https://2j9qacl6k6.execute-api.us-east-1.amazonaws.com/prod/user-profile?user_email=' + email_address;
            const api_response = await fetch(api_url);
            const api_data = await(api_response).json();
            console.log(api_data);
            
            const div_user_info = document.getElementById('profile');
            div_user_info.innerHTML = api_data['body'];
          }
          
        function getUserAttributes() {
          var data = { 
            UserPoolId : _config.cognito.userPoolId,
            ClientId : _config.cognito.clientId
          };
          var userPool = new AmazonCognitoIdentity.CognitoUserPool(data);
          var cognitoUser = userPool.getCurrentUser();

          if (cognitoUser != null) {
            cognitoUser.getSession(function(err, session) {
              if (err) {
                alert(err);
                return;
              }
              //console.log('session validity: ' + session.isValid());
              
              cognitoUser.getUserAttributes(function(err, result) {
                if (err) {
                  console.log(err);
                  return;
                }
                // user email address
                console.log(result[2].getValue());
                getUser(result[2].getValue()) 
              });

            });
          } else {
            console.log("Already signed-out")
          }
        }

.. raw:: html

  <div style="text-align: center; margin-bottom: 2em;">
    <iframe width="560" height="315" src="https://www.youtube.com/embed/IBfbIfa1YFcxxxx" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen>
    </iframe>
  </div>
