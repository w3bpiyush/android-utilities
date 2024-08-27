# GoogleAuthUtils - Utility Class for Google Sign-In in Android

The `GoogleAuthUtils` class simplifies the integration of Google Sign-In using Firebase Authentication in Android applications. This utility class handles the entire Google Sign-In process, from initiating the sign-in intent to authenticating the user with Firebase and saving their details in Firestore.

## Features

- **Easy Integration**: Simplifies Google Sign-In and Firebase Authentication.
- **User Data Management**: Automatically stores user information in Firestore upon successful sign-in.
- **Callback Handling**: Provides success and failure callbacks for handling sign-in results.

## Setup

1. **Add Dependencies**

   Ensure you have the necessary dependencies in your `build.gradle` file:

   ```groovy
   implementation 'com.google.firebase:firebase-auth:latest-version'
   implementation 'com.google.firebase:firebase-firestore:latest-version'
   implementation 'com.google.android.gms:play-services-auth:latest-version'
   ```

2. **Configure Google Sign-In**
   Add your OAuth 2.0 Client ID to your `strings.xml` file:
   ```
   <string name="default_web_client_id">YOUR_CLIENT_ID</string>
   ```

## Usage

1. Create an instance of `GoogleAuthUtils` and initialize it with the current `Activity`:
   ```
   GoogleAuthUtils googleAuthUtils = new GoogleAuthUtils(this);
   ```
2. To start the sign-in process, call the `signIn` method:
   ```
   btnGoogleLogin.setOnClickListener(v -> googleAuthUtils.signIn(this));
   ```
3. Override `onActivityResult` to handle the result of the sign-in intent:

   ```
   @Override
   protected void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
    super.onActivityResult(requestCode, resultCode, data);
    if (requestCode == GoogleAuthUtils.RC_SIGN_IN) {
        googleAuthUtils.handleSignInResult(data, new GoogleAuthUtils.GoogleSignInCallback() {
            @Override
            public void onSuccess(FirebaseUser user) {
                // Handle successful sign-in here
            }

            @Override
            public void onFailure(Exception e) {
                // Handle sign-in failure here
            }
        });
      }
    }
   ```

4. The `GoogleAuthUtils` class manages the Google Sign-In process and Firebase Authentication. Here's the implementation:

- Create utils file and inside that create `utils/GoogleAuthUtils` and paste this below code

  ```
  package com.example.googlesigninauth.utils;

  import android.app.Activity;
  import android.content.Intent;
  import android.util.Log;
  import com.google.android.gms.auth.api.signin.GoogleSignIn;
  import com.google.android.gms.auth.api.signin.GoogleSignInAccount;
  import com.google.android.gms.auth.api.signin.GoogleSignInClient;
  import com.google.android.gms.auth.api.signin.GoogleSignInOptions;
  import com.google.android.gms.common.api.ApiException;
  import com.google.android.gms.tasks.Task;
  import com.google.firebase.auth.AuthCredential;
  import com.google.firebase.auth.FirebaseAuth;
  import com.google.firebase.auth.FirebaseUser;
  import com.google.firebase.auth.GoogleAuthProvider;
  import com.google.firebase.firestore.FirebaseFirestore;

  import com.example.googlesigninauth.R;

  public class GoogleAuthUtils {
   private static final String TAG = "GoogleSignInUtil";
   public static final int RC_SIGN_IN = 100;
   private GoogleSignInClient googleSignInClient;
   private FirebaseAuth firebaseAuth;

   public interface GoogleSignInCallback {
       void onSuccess(FirebaseUser user);
       void onFailure(Exception e);
   }

   public GoogleAuthUtils(Activity activity) {
       this.firebaseAuth = FirebaseAuth.getInstance();

       // Configure Google Sign-In options
       GoogleSignInOptions gso = new GoogleSignInOptions.Builder(GoogleSignInOptions.DEFAULT_SIGN_IN)
               .requestIdToken(activity.getString(R.string.default_web_client_id))
               .requestEmail()
               .build();

       // Build a GoogleSignInClient with the options specified by gso
       this.googleSignInClient = GoogleSignIn.getClient(activity, gso);
   }

   public void signIn(Activity activity) {
       Intent signInIntent = googleSignInClient.getSignInIntent();
       activity.startActivityForResult(signInIntent, RC_SIGN_IN);
   }

   public void handleSignInResult(Intent data, GoogleSignInCallback callback) {
       Task<GoogleSignInAccount> task = GoogleSignIn.getSignedInAccountFromIntent(data);
       try {
           GoogleSignInAccount account = task.getResult(ApiException.class);
           if (account != null) {
               firebaseAuthWithGoogle(account, callback);
           }
       } catch (ApiException e) {
           Log.e(TAG, "Google Sign-In failed: " + e.getLocalizedMessage());
           callback.onFailure(e);
       }
   }

   private void firebaseAuthWithGoogle(GoogleSignInAccount account, GoogleSignInCallback callback) {
       AuthCredential credential = GoogleAuthProvider.getCredential(account.getIdToken(), null);
       firebaseAuth.signInWithCredential(credential)
               .addOnCompleteListener(task -> {
                   if (task.isSuccessful()) {
                       FirebaseUser user = firebaseAuth.getCurrentUser();
                       // save user to database
                   } else {
                       callback.onFailure(task.getException());
                   }
               });
      }
     }
  ```

## Contact

For any questions or issues related to the `GoogleAuthUtils` class, please reach out to:

- **Email**: [w3bpiyush@gmail.com](mailto:w3bpiyush@gmail.com)
- **GitHub**: [github.com/w3bpiyush](https://github.com/w3bpiyush)
