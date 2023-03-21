# Week 3 — Decentralized Authentication

## Amazon Cognito
**Amazon Cognito** is a cloud-based authentication, authorization, and user management service provided by Amazon Web Services (AWS). It is designed to help developers build web and mobile applications that require user sign-up, sign-in, and access control.

With Amazon Cognito, developers can easily add authentication and user management to their applications, and enable their users to sign-up and sign-in with a range of identity providers such as Amazon, Facebook, Google, and others. Additionally, developers can use Amazon Cognito to manage user profiles, including attributes like phone numbers, email addresses, and user preferences.

Amazon Cognito also provides features to help developers control access to their resources, such as providing temporary credentials to access AWS services, or creating fine-grained access control policies to restrict user access to specific application resources.

*Overall, Amazon Cognito simplifies the development process for web and mobile applications that require user authentication and management, and provides a secure and scalable solution for managing user identities and access control.
There are several similar services in the market that provide authentication, authorization, and user management functionalities similar to Amazon Cognito. Some examples include:*

 *1. Auth0: Auth0 is a cloud-based identity platform that offers user authentication, authorization, and user management services for web, mobile, and IoT applications. It provides features such as social identity providers, multi-factor authentication, and customizable login pages.*
*2. Okta: Okta is a cloud-based identity and access management platform that offers features such as single sign-on, multi-factor authentication, and user provisioning. It supports a range of identity providers and offers integration with enterprise systems such as Active Directory.*
*3. Firebase Authentication: Firebase Authentication is a cloud-based service provided by Google that offers user authentication and authorization for web and mobile applications. It supports a range of authentication providers and offers features such as email verification and custom user attributes.*
*4. Azure Active Directory: Azure Active Directory is a cloud-based identity and access management service provided by Microsoft. It offers features such as single sign-on, multi-factor authentication, and access control policies for enterprise applications.*

Example of Amzon Cognito Usecase Scenario
Serverless application authentication: A developer builds a serverless application using AWS Lambda functions and API Gateway. By using Amazon Cognito, the developer can secure their application's endpoints and ensure that only authenticated users can access the application's resources.

## Implenting Amazon Cognito into our Cruddur Application
### creating User pools
I cretaed a user pool from the AWS console and followed the prompts illlustrated in the videos.
### Configure Amplify
**Backend as a Service (BaaS)** is a cloud computing model where third-party vendors provide backend services to application developers. This means that developers do not need to build and maintain their own backend infrastructure and can instead rely on the BaaS provider to handle backend services such as data storage, authentication, and push notifications.
In the case of AWS Amplify, the BaaS offerings include services such as:

1. User authentication: Amplify provides user authentication services that enable developers to add sign-up, sign-in, and user profile management capabilities to their applications without having to build and maintain their own authentication infrastructure.
2. Database management: Amplify provides managed database services such as Amazon DynamoDB and Amazon Aurora Serverless that allow developers to store and manage data in the cloud without having to set up and maintain their own database infrastructure.
3. File storage: Amplify provides file storage services such as Amazon S3 and Amazon Glacier that enable developers to store and manage files in the cloud without having to build and maintain their own file storage infrastructure.

Configuring AWS Amplify
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
Remain instructions are already implemented
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
Encountered a null pointer error due to jwt tokens were not configured. To acheive a sign in visual as below create the user using the cosole with the follwoing AWS Amplify command
```aws
aws cognito-idp admin-set-user-password 
  --username lina 
  --password Testing1234! 
  --user-pool-id us-east-1_iTi7xngE0 
  --permanent
```
**insert image
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
**insertimage
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
**insert image
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
**insert image

