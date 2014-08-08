layer-parse-module
==================
To incorporate Layer Authentication with Parse user management in your application, you will need to use Parse Cloud Code along with this layer-parse-module. This module takes care of generating a Layer Identity Token, and only requires you to implement a few lines of code.


##Installation
If you haven't already, install the Parse command line tool that will let you manage the Parse Cloud Code linked to your Parse application by following the directions outlined [here](https://parse.com/docs/cloud_code_guide).

To get the Layer Module, clone this repo, and place it in your Parse Cloud Code Directory in the cloud folder:

    https://github.com/layerhq/layer-parse-module.git
    
##Usage


###Creating your Cloud Function
To use this module in your Parse Cloud Code, you have to require the module and initialize it with the proper IDs and keys. 
  1. Start by navigating to the keys folder in the repo you just cloned, and open the layer-key.js file. Copy the Private       Key generated through the Layer Developer Portal into this file and save it. 
  2. To require this module, open your main.js file and include this code at the top:
  
        ```javascript
	var fs = require('fs');
        var client = require('cloud/layer-module.js');
        ```
        
  3. Next you must initialize the instance of this module with the proper Provider ID and Key ID generated in your Layer        Developer Portal:
  
  	```javascript
        var layerProviderID = 'YOUR-PROVIDER ID-HERE';
        var layerKeyID = 'YOUR-KEY ID-HERE';
        var privateKey = fs.readFileSync('cloud/keys/layer-key.js');
        client.initialize(layerProviderID, layerKeyID, privateKey);
        ```
        
  4. Finally, you must create Parse Cloud fundtion to call the layerIdentityToke function in the module. Your Cloud function will look something like this:
  
  	```javascript
        Parse.Cloud.define("generateToken", function(request, response) {
	          var userID = request.params.userID;
	          var nonce = request.params.nonce
	          if (!userID) throw new Error('Missing userID parameter');
	          if (!nonce) throw new Error('Missing nonce parameter');
            response.success(layer.layerIdentityToken(userID, nonce));
        });
	```

###Calling your Cloud Function
Once you have created the Cloud function with the layer-parse-module, you must call this function from your application and pass it the appropriate parameters (the userID and a nonce). The userID you are looking for is the objectID of the Parse User.

####iOS

    // Request an authentication nonce from Layer
    [layerClient requestAuthenticationNonceWithCompletion:^(NSString *nonce, NSError *error) {
        NSLog(@"Authentication nonce %@", nonce);
       
        //Upon reciept of nonce, post to your backend and acquire a Layer identityToken  
        PFUser *user = [PFUser currentUser];
        NSString *userID  = user.objectId;
        PFCloud callFunctionInBackground:@"generateToken"
                           withParameters:@{@"nonce" : nonce,
                                            @"userID" : userID}
                                    block:^(NSString *token, NSError *error) {
            if (error) {
                NSLog(@"Parse Cloud function failed to be called to generate token with error: %@", error);
            }
            else{
                [self.layerClient authenticateWithIdentityToken:token completion:^(NSString *authenticatedUserID, NSError                  *error) {
                    if (error) {
                        NSLog(@"Parse User failed to authenticate with token with error: %@", error);
                    }
                    else{
                        NSLog(@"Parse User authenticated with Layer Identity Token");
                    }
                }];
            }
        }];
    }];