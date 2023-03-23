# Week 3 — Decentralized Authentication

## Amazon Cognito
**Amazon Cognito** is a cloud-based authentication, authorization, and user management service provided by Amazon Web Services (AWS). It is designed to help developers build web and mobile applications that require user sign-up, sign-in, and access control.</br>

With Amazon Cognito, developers can easily add authentication and user management to their applications, and enable their users to sign-up and sign-in with a range of identity providers such as Amazon, Facebook, Google, and others. Additionally, developers can use Amazon Cognito to manage user profiles, including attributes like phone numbers, email addresses, and user preferences.</br>

Amazon Cognito also provides features to help developers control access to their resources, such as providing temporary credentials to access AWS services, or creating fine-grained access control policies to restrict user access to specific application resources.</br>

*Overall, Amazon Cognito simplifies the development process for web and mobile applications that require user authentication and management, and provides a secure and scalable solution for managing user identities and access control.
There are several similar services in the market that provide authentication, authorization, and user management functionalities similar to Amazon Cognito. Some examples include:*

  * Auth0: Auth0 is a cloud-based identity platform that offers user authentication, authorization, and user management services for web, mobile, and IoT applications. It provides features such as social identity providers, multi-factor authentication, and customizable login pages.
  * Okta: Okta is a cloud-based identity and access management platform that offers features such as single sign-on, multi-factor authentication, and user provisioning. It supports a range of identity providers and offers integration with enterprise systems such as Active Directory.
  * Firebase Authentication: Firebase Authentication is a cloud-based service provided by Google that offers user authentication and authorization for web and mobile applications. It supports a range of authentication providers and offers features such as email verification and custom user attributes.
  * Azure Active Directory: Azure Active Directory is a cloud-based identity and access management service provided by Microsoft. It offers features such as single sign-on, multi-factor authentication, and access control policies for enterprise applications.

Example of Amazon Cognito Usecase Scenario</br>
    - Serverless application authentication: A developer builds a serverless application using AWS Lambda functions and API Gateway. By using Amazon Cognito, the developer can secure their application's endpoints and ensure that only authenticated users can access the application's resources.

## Implenting Amazon Cognito into our Cruddur Application
### creating User pools
I created a user pool from the AWS console and followed the prompts illlustrated in the videos.
### Configure Amplify
**Backend as a Service (BaaS)** is a cloud computing model where third-party vendors provide backend services to application developers. This means that developers do not need to build and maintain their own backend infrastructure and can instead rely on the BaaS provider to handle backend services such as data storage, authentication, and push notifications.</br>
In the case of AWS Amplify, the BaaS offerings include services such as:
1. User authentication: Amplify provides user authentication services that enable developers to add sign-up, sign-in, and user profile management capabilities to their applications without having to build and maintain their own authentication infrastructure.
2. Database management: Amplify provides managed database services such as Amazon DynamoDB and Amazon Aurora Serverless that allow developers to store and manage data in the cloud without having to set up and maintain their own database infrastructure.
3. File storage: Amplify provides file storage services such as Amazon S3 and Amazon Glacier that enable developers to store and manage files in the cloud without having to build and maintain their own file storage infrastructure.

Configuring AWS Amplify</br>
We need to hook up our cognito pool to our code in the `App.js`
```js
import { Amplify } from 'aws-amplify';

Amplify.configure({
  "AWS_PROJECT_REGION": process.env.REACT_APP_AWS_PROJECT_REGION,
  "aws_cognito_identity_pool_id": process.env.REACT_APP_AWS_COGNITO_IDENTITY_POOL_ID,
  "aws_cognito_region": process.env.REACT_APP_AWS_COGNITO_REGION,
  "aws_user_pools_id": process.env.REACT_APP_AWS_USER_POOLS_ID,
  "aws_user_pools_web_client_id": process.env.REACT_APP_CLIENT_ID,
  "oauth": {},
  Auth: {
    // We are not using an Identity Pool
    // identityPoolId: process.env.REACT_APP_IDENTITY_POOL_ID, // REQUIRED - Amazon Cognito Identity Pool ID
    region: process.env.REACT_APP_AWS_PROJECT_REGION,           // REQUIRED - Amazon Cognito Region
    userPoolId: process.env.REACT_APP_AWS_USER_POOLS_ID,         // OPTIONAL - Amazon Cognito User Pool ID
    userPoolWebClientId: process.env.RREACT_APP_CLIENT_ID,   // OPTIONAL - Amazon Cognito Web Client ID (26-char alphanumeric string)
  }
});
```
Add the following on the `docker-compose.yml`  file under the frontend variables
```yml
AWS_DEFAULT_REGION: "${AWS_DEFAULT_REGION}"
      AWS_ACCESS_KEY_ID: "${AWS_ACCESS_KEY_ID}"
      AWS_SECRET_ACCESS_KEY: "${AWS_SECRET_ACCESS_KEY}"
      ROLLBAR_ACCESS_TOKEN: "${ROLLBAR_ACCESS_TOKEN}"
      AWS_COGNITO_USER_POOL_ID: "us-east-1_kfbRA48s2"
      AWS_COGNITO_USER_POOL_CLIENT_ID: "2fd3d30tbmjelrafi89ikonmt7" 
```
## Implement Custom Signin Page
#### Conditionally show components based on logged in or logged out
Inside `HomeFeedPage.js`
```js
 // check if we are authenicated
const checkAuth = async () => {
  Auth.currentAuthenticatedUser({
    // Optional, By default is false. 
    // If set to true, this call will send a 
    // request to Cognito to get the latest user data
    bypassCache: false 
  })
  .then((user) => {
    console.log('user',user);
    return Auth.currentAuthenticatedUser()
  }).then((cognito_user) => {
      setUser({
        display_name: cognito_user.attributes.name,
        handle: cognito_user.attributes.preferred_username
      })
  })
  .catch((err) => console.log(err));
};
```
Remain instructions are already implemented</br>
Edit `ProfileInfo.js` to add the following
```js
import { Auth } from 'aws-amplify';
//I replaced the below code with the current one using cookies
const signOut = async () => {
    try {
        await Auth.signOut({ global: true });
        window.location.href = "/"
    } catch (error) {
        console.log('error signing out: ', error);
    }
  }
```
Add the following to the `signin,js` page - replace current code using cookies formaintaining session
```js
import { Auth } from 'aws-amplify';
const onsubmit = async (event) => {
    setErrors('')
    event.preventDefault();
   
      Auth.signIn(email, password)
        .then(user => {
          console.log('user',user)
          localStorage.setItem("access_token", user.signInUserSession.accessToken.jwtToken)
          window.location.href = "/"
        })
        .catch(error => { 
          if (error.code == 'UserNotConfirmedException') {
            window.location.href = "/confirm"
          }
          setErrors(error.message)
        });
    return false
  }
```
Encountered a null pointer error due to jwt tokens were not configured. To acheive a sign in visual as below create the user using the console with the follwoing AWS Amplify command
```aws
aws cognito-idp admin-set-user-password 
  --username lina 
  --password Testing1234! 
  --user-pool-id us-east-1_iTi7xngE0 
  --permanent
```
![Signin Page working with manually created user](https://github.com/lkiunga/aws-bootcamp-cruddur-2023/blob/main/journal/assets/Week3-signin-page-working-with-manually-created-user.png)
## Implement Custom Signup Page
Delete the manually created user and setup a Sign Up page from the AWS Cognito Console page
update the `SignupPage.js` file with the following code
```js
import { Auth } from 'aws-amplify';

// replace the folllowing function with these codes
const onsubmit = async (event) => {
  event.preventDefault();
  setErrors('')
  try {
      const { user } = await Auth.signUp({
        username: email,
        password: password,
        attributes: {
            name: name,
            email: email,
            preferred_username: username,
        },
        autoSignIn: { // optional - enables auto sign in after user is confirmed
            enabled: true,
        }
      });
      console.log(user);
      window.location.href = `/confirm?email=${email}`
  } catch (error) {
      console.log(error);
      setErrors(error.message)
  }
  return false
}
```
![SignUp page with the additional Parameter](https://github.com/lkiunga/aws-bootcamp-cruddur-2023/blob/main/journal/assets/week3-sign-up-page-working.png)
![Verified user after SIGN up](https://github.com/lkiunga/aws-bootcamp-cruddur-2023/blob/main/journal/assets/Week3-sign-up-image-confirmed.png)
update `ConfirmationPage.js` page with the following code
```js
const resend_code = async (event) => {
    setErrors('')
    try {
      await Auth.resendSignUp(email);
      console.log('code resent successfully');
      setCodeSent(true)
    } catch (err) {
      // does not return a code
      // does cognito always return english
      // for this to be an okay match?
      console.log(err)
      if (err.message == 'Username cannot be empty'){
        setErrors("You need to provide an email in order to send Resend Activiation Code")   
      } else if (err.message == "Username/client id combination not found."){
        setErrors("Email is invalid or cannot be found.")   
      }
    }
  }

  const onsubmit = async (event) => {
    event.preventDefault();
    setErrors('')
    try {
      await Auth.confirmSignUp(email, code);
      window.location.href = "/"
    } catch (error) {
      setErrors(error.message)
    }
    return false
  }
```
![]()
update `RecoveryPage.js` page with the following code
```js
import { Auth } from 'aws-amplify';
//replace the following functions
 const onsubmit_send_code = async (event) => {
    event.preventDefault();
    setErrors('')
    Auth.forgotPassword(username)
    .then((data) => setFormState('confirm_code') )
    .catch((err) => setErrors(err.message) );
    return false
  }
  const onsubmit_confirm_code = async (event) => {
    event.preventDefault();
    setErrors('')
    if (password == passwordAgain){
      Auth.forgotPasswordSubmit(username, code, password)
      .then((data) => setFormState('success'))
      .catch((err) => setErrors(err.message) );
    } else {
      setErrors('Passwords do not match')
    }
    return false
  }
```
![Recovery Email Working](https://github.com/lkiunga/aws-bootcamp-cruddur-2023/blob/main/journal/assets/week3-resendActivation-email.png)
![Verified working accounts for users](https://github.com/lkiunga/aws-bootcamp-cruddur-2023/blob/main/journal/assets/Week3-sign-up-image-confirmed.png)
![Successful Login for user](https://github.com/lkiunga/aws-bootcamp-cruddur-2023/blob/main/journal/assets/week3-user-account-working-succesful.png)
## Verify JWT token server side
A JSON Web Token, or JWT, is an open standard for securely creating and sending data between two parties, usually a client and a server.</br>
JWTs are usually used to manage user sessions on a website. While they're an important part of the token based authentication process, **JWTs themselves are used for authorization, not authentication.**</br>
![JWT working](https://github.com/lkiunga/aws-bootcamp-cruddur-2023/blob/main/journal/assets/week3-how-JWT-works.png)
For better understanding I refernced this material
[How to Sign and Validate JSON Web Tokens – JWT Tutorial](https://www.freecodecamp.org/news/how-to-sign-and-validate-json-web-tokens/#:~:text=This%20process%20is%20called%20authorization,it%20sends%20back%20a%20response.)</br>
#### For the CRUDDUR APP to verify the JWT token on the server side</br>
**1. Add the following into the `requirements.txt` file and install the new requirements**
```
Flask-AWSCognito
```
**2.Create a new folder called `lib` and in it create a file called `cognito_jwt_token.py`.**</br>
- copy the following code into it 
```python
import time
import requests
from jose import jwk, jwt
from jose.exceptions import JOSEError
from jose.utils import base64url_decode

class FlaskAWSCognitoError(Exception):
  pass

class TokenVerifyError(Exception):
  pass

def extract_access_token(request_headers):
    access_token = None
    auth_header = request_headers.get("Authorization")
    if auth_header and " " in auth_header:
        _, access_token = auth_header.split()
    return access_token

class CognitoJwtToken:
    def __init__(self, user_pool_id, user_pool_client_id, region, request_client=None):
        self.region = region
        if not self.region:
            raise FlaskAWSCognitoError("No AWS region provided")
        self.user_pool_id = user_pool_id
        self.user_pool_client_id = user_pool_client_id
        self.claims = None
        if not request_client:
            self.request_client = requests.get
        else:
            self.request_client = request_client
        self._load_jwk_keys()


    def _load_jwk_keys(self):
        keys_url = f"https://cognito-idp.{self.region}.amazonaws.com/{self.user_pool_id}/.well-known/jwks.json"
        try:
            response = self.request_client(keys_url)
            self.jwk_keys = response.json()["keys"]
        except requests.exceptions.RequestException as e:
            raise FlaskAWSCognitoError(str(e)) from e

    @staticmethod
    def _extract_headers(token):
        try:
            headers = jwt.get_unverified_headers(token)
            return headers
        except JOSEError as e:
            raise TokenVerifyError(str(e)) from e

    def _find_pkey(self, headers):
        kid = headers["kid"]
        # search for the kid in the downloaded public keys
        key_index = -1
        for i in range(len(self.jwk_keys)):
            if kid == self.jwk_keys[i]["kid"]:
                key_index = i
                break
        if key_index == -1:
            raise TokenVerifyError("Public key not found in jwks.json")
        return self.jwk_keys[key_index]

    @staticmethod
    def _verify_signature(token, pkey_data):
        try:
            # construct the public key
            public_key = jwk.construct(pkey_data)
        except JOSEError as e:
            raise TokenVerifyError(str(e)) from e
        # get the last two sections of the token,
        # message and signature (encoded in base64)
        message, encoded_signature = str(token).rsplit(".", 1)
        # decode the signature
        decoded_signature = base64url_decode(encoded_signature.encode("utf-8"))
        # verify the signature
        if not public_key.verify(message.encode("utf8"), decoded_signature):
            raise TokenVerifyError("Signature verification failed")

    @staticmethod
    def _extract_claims(token):
        try:
            claims = jwt.get_unverified_claims(token)
            return claims
        except JOSEError as e:
            raise TokenVerifyError(str(e)) from e

    @staticmethod
    def _check_expiration(claims, current_time):
        if not current_time:
            current_time = time.time()
        if current_time > claims["exp"]:
            raise TokenVerifyError("Token is expired")  # probably another exception

    def _check_audience(self, claims):
        # and the Audience  (use claims['client_id'] if verifying an access token)
        audience = claims["aud"] if "aud" in claims else claims["client_id"]
        if audience != self.user_pool_client_id:
            raise TokenVerifyError("Token was not issued for this audience")

    def verify(self, token, current_time=None):
        """ https://github.com/awslabs/aws-support-tools/blob/master/Cognito/decode-verify-jwt/decode-verify-jwt.py """
        if not token:
            raise TokenVerifyError("No token provided")

        headers = self._extract_headers(token)
        pkey_data = self._find_pkey(headers)
        self._verify_signature(token, pkey_data)

        claims = self._extract_claims(token)
        self._check_expiration(claims, current_time)
        self._check_audience(claims)

        self.claims = claims 
        return claims
```
*  This code provides a Flask extension for verifying JWT tokens issued by AWS Cognito User Pools.</br>
*  The **CognitoJwtToken** class is responsible for loading the public JWK keys from the Cognito User Pool and providing a verify() method to verify the JWT token's signature, expiration, and audience.</br>
*  The **verify()** method takes a JWT token as input and verifies its signature using the public key from the JWK keys. It then extracts the claims from the JWT token and checks for expiration and audience (the client ID of the user pool). If the token passes all these checks, the claims are stored in the **self.claims** attribute of the instance and returned.

* The **extract_access_token** function is a utility function to extract the access token from the request headers.

* The **TokenVerifyError and FlaskAWSCognitoError** exceptions are defined for handling errors during the verification process.</br>

**3. To verify whether the JWT tokens are working, add the folowing code to the `home_activities.py` **

```python
if cognito_user_id != None:
        extra_crud = {
          'uuid': '248959df-3079-4947-b847-9e0892d1bab4',
          'handle':  'Lore',
          'message': 'My dear brother, it the humans that are the problem',
          'created_at': (now - timedelta(hours=1)).isoformat(),
          'expires_at': (now + timedelta(hours=12)).isoformat(),
          'likes': 1042,
          'replies': []
        }
        results.insert(0,extra_crud)

```
The above code checks for the `cognito_user_id`, if provided it will show the above comment on the user feed for the particulat authenticated user.</br>
If Null - the `home_activities.py` will show only the static content approved for all users</br>
![showwing authenitaced session showing the new added activity](https://github.com/lkiunga/aws-bootcamp-cruddur-2023/blob/main/journal/assets/week3-sessions-at-play.png)

**4. Add the following code to the file `app.py`**

```python
# On the imports
from flask_cors import CORS, cross_origin
#It uses Flask-CORS to allow cross-origin resource sharing 
from lib.cognito_jwt_token import CognitoJwtToken, extract_access_token, TokenVerifyError
# after app = Flask(__name__)
#set JWT env variables
cognito_jwt_token = CognitoJwtToken(
  user_pool_id=os.getenv("AWS_COGNITO_USER_POOL_ID"), 
  user_pool_client_id=os.getenv("AWS_COGNITO_USER_POOL_CLIENT_ID"),
  region=os.getenv("AWS_DEFAULT_REGION")
)
#set CORS - honestly struggling to understnad the concept on CORS
cors = CORS(
  app, 
  resources={r"/api/*": {"origins": origins}},
  headers=['Content-Type', 'Authorization'], 
  expose_headers='Authorization',
  methods="OPTIONS,GET,HEAD,POST"
)
#rebuild this fuction - still in implemtation stages - 
#we can prove JWT is happening but we need to pass  the message securely
def data_home():
  access_token = extract_access_token(request.headers)
  try:
    claims = cognito_jwt_token.verify(access_token)
    # authenicatied request
    app.logger.debug("authenicated")
    app.logger.debug(claims)
    app.logger.debug(claims['username'])
    data = HomeActivities.run(cognito_user_id=claims['username'])
  except TokenVerifyError as e:
    # unauthenicatied request
    app.logger.debug(e)
    app.logger.debug("unauthenicated")
    data = HomeActivities.run()
  return data, 200
```
![JWT verification in process](https://github.com/lkiunga/aws-bootcamp-cruddur-2023/blob/main/journal/assets/week3-shows-session-play-with-cognito-id.png)
 - - - -
# Homework Challenges
## Improving UI Contrast and Implementing CSS Variables for Theming 
Create CSS variables to enable uniform changes on the theming of the application for better visibility on windows machine.</br>
- **create css varables on the `index.css` page and update the codes of the background section bellow**
```
:root {
  --bg: rgb(61,13,123);
  --fg: rgb(8,1,14);

  --field-border: rgb(255,255,255,0.29);
  --field-border-focus: rgb(149,0,255,1);
  --field-bg: rgb(31,31,31);
}
```
-update the code of the following files 
* Desktopsidebar.css
* Signuppage.css
* SignInpage.css
* Activityitem.css
* joinsection.css
* search.css

 - - - -






