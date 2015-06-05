# AdManager
Advertisement Manager iOS / Android API Documentation

#1. API Information
#1.1. List Accounts Service

    /accounts/:username/:hash

Example API CALL
    api.admanager.crenno.com/rpc.php/accounts/crennotech/74e749b04c50cba2edc0eb507d5e4ddf

Salt value for hash mechanism:

#1.2. Application Authentication and Advertisements Service

    /authenticate/:id/:date/:hash

Example API CALL
    api.admanager.crenno.com/rpc.php/authenticate/3991/782917/3a52a76876fd463ea344f240e73dcd92

Salt value for hash mechanism:

#2. Android Applications API Integration

#2.1. Prequisites

Add static default values into strings.xml of your Project, under res/values

Warning:
In this documentation, we assumed that you’ve already installed your existing admob account into your Project, and it works well. There is no information about how to enable Admob account into your Android application in this document.

    <string name="app_id">your-admanager-app-id</string>
    <string name="banner_ad_unit_id">ca-app-pub-xxx</string>
    <string name="inter_ad_unit_id">ca-app-pub-xxx</string>


Change AdMob.xml
    <?xml version="1.0" encoding="utf-8"?>
    <FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:ads="http://schemas.android.com/apk/res-auto"
        android:id="@+id/reklam"
        android:layout_width="fill_parent"
        android:layout_height="50dp"
        android:layout_marginBottom="0dip">	   
    </FrameLayout>
  
#2.2. Application Defaults
For he first time app usage, we should set the default admob keys into the application shared settings for offline usage or connection problems. 

    SharedPreferences settings = getSharedPreferences(context.getPackageName() + "_preferences", 0);
    //Context should be your activity name you use in, ex. MainActivity.this
    boolean silent = settings.getBoolean("silentMode", false);       
    if(!silent){
        SharedPreferences.Editor editor = settings.edit();
        editor.putBoolean("silentMode", true);
        editor.putString("StandardAdID",getString(R.string.banner_ad_unit_id));
        editor.putString("InterstatialAdID",getString(R.string.inter_ad_unit_id));
        editor.commit();
    }

#2.3. Web Service Call Class
Just for information, please change your outer class name, dictated red below. In this code, we used this inner class inside of SplashActivity. Just add it into similar entrance class of your Project, and change the name in this class.

    class RequestTask extends AsyncTask<String, String, String>{

        @Override
        protected void onPreExecute() {
            super.onPreExecute();
            //get sharedPreferences here
        }

        @Override
        protected String doInBackground(String... uri) {
            HttpClient httpclient = new DefaultHttpClient();
            HttpResponse response;
            String responseString = null;
            try {
                response = (HttpResponse) httpclient.execute(new HttpGet(uri[0]));
                if(response.getStatusCode() == HttpStatus.SC_OK){
                    ByteArrayOutputStream out = new ByteArrayOutputStream();
                    ((org.apache.http.HttpResponse) response).getEntity().writeTo(out);
                    responseString = out.toString();
                    out.close();
                } else{
                    //Closes the connection.
                    ((org.apache.http.HttpResponse) response).getEntity().getContent().close();
                    throw new IOException(response.getReasonPhrase());
                }
            } catch (ClientProtocolException e) {
                //TODO Handle problems..
            } catch (IOException e) {
                //TODO Handle problems..
            } catch (Exception e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            }
            return responseString;
        }

        @Override
        protected void onPostExecute(String result) {
            super.onPostExecute(result);
            try {
                JSONArray jArray = new JSONArray(result);    
                for(int i=0; i < jArray.length(); i++) {
                    JSONObject jObject = jArray.getJSONObject(i);
                    String advertisementID = "ca-app-pub-"+jObject.getString("StandardAdID");
                    String interstatialID = "ca-app-pub-"+jObject.getString("InterstatialAdID");
                    SharedPreferences settings = PreferenceManager.getDefaultSharedPreferences(SplashActivity.this);
                    SharedPreferences.Editor editor = settings.edit();
                    editor.putString("StandardAdID",advertisementID);
                    editor.putString("InterstatialAdID",interstatialID);
                    editor.commit();

                } // End Loop
            } catch (JSONException e) {
                Log.e("JSONException", "Error: " + e.toString());
            }
        }
    }

2.4. Call web service inside of onCreate of your project’s Entrance Class
Just create a new date and prepare hash, than call the webservice

    Date date = new Date();
    String formattedDate = new SimpleDateFormat("yyyy-MM-dd").format(date);
    String challange = getString(R.string.app_secret)+""+formattedDate+""+getString(R.string.app_id)+"";

    try {
    new RequestTask().execute("http://api.admanager.crenno.com/rpc.php/authenticate/"+ getString( R.string.app_id)+"/"+formattedDate+"/"+MD5(challange));

    } catch (UnsupportedEncodingException e) {
        e.printStackTrace();
    }

2.5. JAVA Android MD5 Hash function
    public String MD5(String md5) throws UnsupportedEncodingException {
    try {
        java.security.MessageDigest md = java.security.MessageDigest.getInstance("MD5");
        byte[] array = md.digest(md5.getBytes("UTF-8"));
        StringBuffer sb = new StringBuffer();
        for (int i = 0; i < array.length; ++i) {
            sb.append(Integer.toHexString((array[i] & 0xFF) | 0x100).substring(1,3));
        }
        return sb.toString();
    } catch (java.security.NoSuchAlgorithmException e) {
    }
        return null;
    }

#2.6. Banners and Interstatials in your activities

Add Variable Advertisement Banners inside of your other Actitivites

    SharedPreferences settings = getSharedPreferences(context.getPackageName() + "_preferences", 0);     
    ////Context should be your activity name you use in, ex. MainActivity.this

    FrameLayout reklam = (FrameLayout)findViewById(R.id.reklam);
    AdView adView = new AdView(HaberlerActivity.this);
    adView.setAdUnitId(settings.getString("StandardAdID", getString(R.string.banner_ad_unit_id)));
    adView.setAdSize(AdSize.SMART_BANNER);
    reklam.addView(adView);
    AdRequest adRequest = new AdRequest.Builder().build();
    adView.loadAd(adRequest);
    // Create the interstitial.
    interstitial = new InterstitialAd(this);  
    interstitial.setAdUnitId(settings.getString("InterstatialAdID", getString(R.string.inter_ad_unit_id)));

#3. iOS Applications API Integration
#3.1. Defining Parameters

In your AppDelegate, or Constants Header file define static parameters as below

    #define kAdManagerAppID @"8103"
    #define kAdManagerURL @"api.admanager.crenno.com/rpc.php/authenticate"
    #define kAdManagerAppSecret @"==8nH6M!lLQ4+gUX"
    #define kAdManagerPrefix @"ca-app-pub-"
    #define kAdmobBannerKey @"ca-app-pub-XXX"
    #define kAdmobIntersKey @"ca-app-pub-XXX"
	
#3.2. Create AdManagerHelper Header
    //  AdManagerHelper.h
    //
    //  Created by CRENNO MOBIL TEKNOLOJI on 05/06/15.
    //  Copyright (c) 2015 Crenno. All rights reserved.
    //

    #import <Foundation/Foundation.h>
    #import "AFNetworking.h"

    @interface AdManagerHelper : NSObject

    - (void) updateAdUnits;
    - (NSString *) md5:(NSString *) input;
    - (NSString*) getDateTime;

    @end


#3.3. Create AdManagerHelper Class
    //
    //  AdManagerHelper.m
    //
    //  Created by CRENNO MOBIL TEKNOLOJI on 05/06/15.
    //  Copyright (c) 2015 Crenno. All rights reserved.
    //

    #import <Foundation/Foundation.h>
    #import "AdManagerHelper.h"
    #import "AFNetworking.h"
    #import "AppDelegate.h"
    #import <CommonCrypto/CommonDigest.h>

    @implementation AdManagerHelper

    - (void) updateAdUnits
    {

        NSString * params = [NSString stringWithFormat:@"%@%@%@",kAdManagerAppSecret, [self getDateTime], kAdManagerAppID];
        NSString * reqURL = [NSString stringWithFormat:@"%@/%@/%@/%@", kAdManagerURL, kAdManagerAppID, [self getDateTime], [self md5:params]];

        NSURLRequest * request = [NSURLRequest requestWithURL:[NSURL URLWithString:reqURL]];

        AFJSONRequestOperation *operation = [AFJSONRequestOperation JSONRequestOperationWithRequest:request success:^(NSURLRequest *request, NSHTTPURLResponse *response, id JSON)
         {
             NSError *error;
             @try {

                 NSDictionary *resp = [NSJSONSerialization JSONObjectWithData:JSON options: NSJSONReadingMutableContainers error: &error];

                 [[NSUserDefaults standardUserDefaults] setObject:[NSString stringWithFormat:@"%@%@",kAdManagerPrefix,[resp objectForKey:@"StandardAdID"]] forKey:@"StandardAdID"];
                 [[NSUserDefaults standardUserDefaults] synchronize];

                 [[NSUserDefaults standardUserDefaults] setObject:[NSString stringWithFormat:@"%@%@",kAdManagerPrefix,[resp objectForKey:@"InterstatialAdID"]] forKey:@"InterstatialAdID"];
                 [[NSUserDefaults standardUserDefaults] synchronize];
             }
             @catch (NSException *exception) {
                 NSLog(@"Exception: %@ : %@",exception, error);
             }

         } failure:^(NSURLRequest *request, NSHTTPURLResponse *response, NSError *error, id JSON)
         {

         }];
        [operation start];
    }

    - (NSString *) md5:(NSString *) input
    {
        const char *cStr = [input UTF8String];
        unsigned char digest[16];
        CC_MD5( cStr, strlen(cStr), digest );

        NSMutableString *output = [NSMutableString stringWithCapacity:CC_MD5_DIGEST_LENGTH * 2];

        for(int i = 0; i < CC_MD5_DIGEST_LENGTH; i++)
            [output appendFormat:@"%02x", digest[i]];

        return  output;

    }

    - (NSString*) getDateTime {
        NSDateFormatter *formatter;
        NSString        *dateString;

        formatter = [[NSDateFormatter alloc] init];
        [formatter setDateFormat:@"yyyy-MM-dd"];

        dateString = [formatter stringFromDate:[NSDate date]];

        return dateString;
    }

    @end

#3.4. Accessing AdManagerHelper inside of the AppDelegate
    //import first
    #import "AdManagerHelper.h"

    //Add Custom self method
    -(void) configureAdManager
    {
        [AdManagerHelper updateAdUnits];
    }

    //Call custom method inside of the didFinishLaunchingWithOptions method
    - (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
    {
        //…

        [self configureAdManager];

    //…
    }

#3.5. Using Banner ID Dynamically

    if([[NSUserDefaults standardUserDefaults] objectForKey:@"StandardAdID"]!=nil){
		self.bannerView.adUnitID = [[NSUserDefaults standardUserDefaults] objectForKey:@"StandardAdID"];
	}else{
		self.bannerView.adUnitID = kAdmobBannerKey;
	}


#3.6. Using Interstatial ID Dynamically

    if([[NSUserDefaults standardUserDefaults] objectForKey:@"InterstatialAdID"]!=nil){
		self.interstitial.adUnitID = [[NSUserDefaults standardUserDefaults] objectForKey:@"InterstatialAdID"];
	}else{
		self.interstitial.adUnitID = kAdmobIntersKey;

