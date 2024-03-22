### Cristian Domenico Dramisino (12338532)

# Part 1 

## Introduction

The challenge can be accessed using the link https://sas.hackthe.space/cms/files/72dca2ff7be89806c320805128da886f1c6a1589f62f35107c0eb646393287c2/braf.2024S.apk . 

The file that is downloaded is a apk file named `braf.2024S.apk`


## Static analysis

==**The static analysis is used to analyze the app without running it and only looking at the code.**==
So what we will do first is to analyze the content of the apk file using the tool JADX.

Basically we use JADX-GUI to decompile the apk and having an overview of the content of the application such as the source code and all the resources it uses.

![[Pasted image 20240321174813.png]]

### AndroidManifest.xml analysis
<mark style="background: #BBFABBA6;">We know that one of the most important files in an android application is the AndroidManifest.xml </mark>

```xml
<?xml version="1.0" encoding="utf-8"?>  
<manifest xmlns:android="http://schemas.android.com/apk/res/android" android:versionCode="2" android:versionName="2024.SS.0" android:compileSdkVersion="33" android:compileSdkVersionCodename="13" package="wien.secpriv.challenges.braf" platformBuildVersionCode="33" platformBuildVersionName="13">  
    <uses-sdk android:minSdkVersion="30" android:targetSdkVersion="33"/>  
    <uses-permission android:name="android.permission.INTERNET"/>  
    <application android:theme="@style/AppTheme" android:label="@string/app_name" android:icon="@mipmap/braf_launcher" android:name="wien.secpriv.challenges.braf.BobRossApp4Fans" android:allowBackup="true" android:supportsRtl="true" android:extractNativeLibs="false" android:networkSecurityConfig="@xml/network_security_config" android:roundIcon="@mipmap/braf_launcher" android:appComponentFactory="androidx.core.app.CoreComponentFactory">  
        <activity android:name="wien.secpriv.challenges.braf.ImageDialog"/>  
        <activity android:name="wien.secpriv.challenges.braf.GuessingActivity"/>  
        <activity android:name="wien.secpriv.challenges.braf.GalleryActivity"/>  
        <activity android:name="wien.secpriv.challenges.braf.QuoteActivity"/>  
        <activity android:name="wien.secpriv.challenges.braf.RiddleActivity"/>  
        <activity android:name="wien.secpriv.challenges.braf.AboutActivity"/>  
        <activity android:name="wien.secpriv.challenges.braf.MainActivity" android:exported="true">  
            <intent-filter>  
                <action android:name="android.intent.action.MAIN"/>  
                <category android:name="android.intent.category.LAUNCHER"/>  
            </intent-filter>  
        </activity>  
        <meta-data android:name="com.google.android.gms.version" android:value="@integer/google_play_services_version"/>  
        <provider android:name="androidx.startup.InitializationProvider" android:exported="false" android:authorities="wien.secpriv.challenges.braf.androidx-startup">  
            <meta-data android:name="androidx.emoji2.text.EmojiCompatInitializer" android:value="androidx.startup"/>  
            <meta-data android:name="androidx.lifecycle.ProcessLifecycleInitializer" android:value="androidx.startup"/>  
        </provider>  
        <activity android:theme="@style/Theme.PlayCore.Transparent" android:name="com.google.android.play.core.common.PlayCoreDialogWrapperActivity" android:exported="false" android:stateNotNeeded="true"/>  
    </application>  
</manifest>
```

As we can see there are no interesting things in it. 
There is an activity that has the property `exported = "true"` 
```xml
<activity android:name="wien.secpriv.challenges.braf.MainActivity" android:exported="true">  
```
- this means that it could be accessed from other application or in general from the extern.
- **we can take note about it**, maybe it could contain some useful information


### Resources Analysis
Since the AndroidManifest.xml has no other interesting information we could take a look at the resources, starting from the generic folder `values` :
![[Pasted image 20240321173932.png]]

**==In many cases android applications leak sensitive information because developers hardcode this information in the source code or generally in android development into the file ==** `strings.xml`

So it could make sense to study it and try to figure out if there are some useful information in it.

In fact, we could see that there is an interesting string:
```XML
<string name="easyone">FLG_PT1{here_is_already_the_first_one}</string>
```


So, the flag for the part 1 is: `FLG_PT1{here_is_already_the_first_one}`






# Challenge 2

## Static analysis 
We use also here JADX-GUI to decompile the apk and to have access to the source code.

We know by the description that the target for the second flag is for sure the GalleryActivity and all the activities related to it:
```
What's our favourite picture in the Gallery? Find the correct name and get a flag! Though it's a bit more complicated since we ~~accidentally~~ hid part of the flag. Hint: Not everything is the same.

The Flag format is `FLG_PT2{xxx_4567890abcde}`. That's a 12-character, lower-case, hexadecimal suffix.

```

So the first thing that we can do is to study the code of:
- GalleryActivity 
- GuessingActivity

### GalleryActivity.java analizing
In this activity nothing relevant happens. 
The code contained in this class is used to charge all the paintings in the Activity that is shown to the user.


![[Pasted image 20240322172114.png]]


## GuessingActivity.java
This is the most important Activity to analyze in this part of the challenge because it contains the information that can allow us to get the flag.

![[Pasted image 20240322173404.png]]


The idea is to try to figure out which is the painting from which the flag is generated (the "fauvorite one") in order to generate this flag by ourself.


So looking at the code of the file GuessingActivity.java we can notice:
```JAVA
 public void checkGuess(View view) {  
        Toast.makeText(this, verify(((EditText) findViewById(R.id.input_guess)).getText().toString()) ? "Correct!" : "That's not it!", 1).show();  
    }
```
- this snippet of code is responsible to check if the string insert by the user in the textfield is name of the "favourite painting"
- to verify the condition, this method calls another method called verify, passing to it the string insert by the user


Let's get a look at the verify method:
```JAVA 
private boolean verify(String str) {  
        String str2;  
        Log.d(MainActivity.TAG, "got guess: " + str);  
        Class galleryContentByName = GalleryActivity.getGalleryContentByName(str);  
        if (galleryContentByName == null) {  
            return false;  
        }  
        try {  
            str2 = (String) galleryContentByName.getMethod("getID", new Class[0]).invoke(null, new Object[0]);  
        } catch (IllegalAccessException | NoSuchMethodException | InvocationTargetException unused) {  
            Log.e(MainActivity.TAG, "Yeah this should not happen");  
            str2 = "";  
        }  
        return str2.equals("6E3E25FC3EBEE6CDF0B383683B261045D3DD52D5D3C106BD5F11FBCAD01C8285");  
    }
```


Analyzing it we can say that this method:
1. Takes as input a string str (the one used by the user in the textview)
2. It uses the string str to obtain the Class object related to this string via a method of the GalleryActivity class
	1. If the obtained object is null, it returns false
3. It uses reflection to invoke the method getID on the obtained object and stores the result in a string str2
	1. If an exception occurs during reflection, it logs an error message and sets str2 to an empty string
4. It returns true if the value of str2 is equal to a "6E3E25FC3EBEE6CDF0B383683B261045D3DD52D5D3C106BD5F11FBCAD01C8285"

So basically to understand which is the right painting we need to insert the name of a painting such that the return of the method getID() of the java class related to this painting is equal to "6E3E25FC3EBEE6CDF0B383683B261045D3DD52D5D3C106BD5F11FBCAD01C8285"



Now we can search in JADX "getID" and we can notice that each one painting java class has a specific java class in the source code and each one implements this method:
![[Pasted image 20240322174008.png]]

If we look at them we can see that each one implements this method calling the static method of the class GalleryContent, but in two different ways:
```java
public static String getID() {  
        return GalleryContent.computeHash(secret, name);  
    }
```


```java
public static String getID() {  
        return GalleryContent.computeHash(name, secret);  
    }
```



**==So, the method is the same but is called with a different order for the parameters by the different classes.==**
- we have to take this in mind for later


At this point we could take a look to the method GalleryContent.computeHash(String str, String str2):
```java
    public static String computeHash(String str, String str2) {  
        try {  
            MessageDigest messageDigest = MessageDigest.getInstance("SHA-256");  
            messageDigest.update(str.getBytes("UTF-8"));  
            messageDigest.update(str2.getBytes("UTF-8"));  
            return bytesToHex(messageDigest.digest());  
        } catch (UnsupportedEncodingException | NoSuchAlgorithmException unused)         {  
            Log.e(MainActivity.TAG, "Yeah this should not happen");  
            return "";  
        }  
    }


```
- basically it generates a SHA-256 digest
- `bytesToHex` method is used to convert bytes in hex representation



## Painting name discovering

So now we have all the information to try to find out which is the painting that generates an ID equal to 6E3E25FC3EBEE6CDF0B383683B261045D3DD52D5D3C106BD5F11FBCAD01C8285 .

To do this we can try to offline brute force this digest using the same process used by the application source code.

For this purpose we can use a java class that automates the process using a txt file that contains for each class ( representing a painting) :
- for each line the name / secret pair or the secret / name pair depending on how they are used in the computeHash function

Txt file content:
```
CleverButterfliesFeedSlowly QuickBadgersGatherExpectantly  # in this case (name,secret)
RapidShowersInterviewSupposedly BrownShopsSaveInstead  # in this case (secret,name)
PointedCommercialsExtractUneasily IdeologicalPhotographersHangUnfortunately
AloneDebtsDriftLovingly CharacteristicKnowledgesUpdateLast
AdvanceDamsInterviewDouble AvailableJamsRealizeThough
SecureGoodsMoveFurthermore WidePridesContrastBarely
IntensePropagandasAidCompletely ComparableBatteriesSimulateSubstantially
DestructiveRemediesPauseAbroad InsideChaosConsiderOff
SufficientAdviceImpressFrankly RepeatedTransmissionsComplainLow
PerfectEmergenciesStabilizeAstonishingly UltimateFarmingsGeneratePermanently
ExcessiveBooksDiscoverObviously IntermediateIntensitiesSupposeAround
SecondaryMusicalsCountAppropriately PreciseMeatsInterfereFreely
...
```

> NOTE: I upload in TUWEL the java file and the txt file
> The txt file contains only 100 lines because i tested the java "script" each 25 lines and I found the result at line 81, so it had no sense to fill the file with the other values.



Java "script" used to find the correct result :
```java

import java.io.UnsupportedEncodingException;
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;
import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;


/* loaded from: classes.dex */
public class DigestCreator  {
    

     public static String bytesToHex(byte[] bArr) {
        char[] cArr = new char[bArr.length * 2];
        char[] HEX_ARRAY = "0123456789ABCDEF".toCharArray();
        for (int i2 = 0; i2 < bArr.length; i2++) {
            int i3 = bArr[i2] & 255;
            int i4 = i2 * 2;
            char[] cArr2 = HEX_ARRAY;
            cArr[i4] = cArr2[i3 >>> 4];
            cArr[i4 + 1] = cArr2[i3 & 15];
        }
        return new String(cArr);
    }

    public static String computeHash(String str, String str2) {
        try {
            MessageDigest messageDigest = MessageDigest.getInstance("SHA-256");
            messageDigest.update(str.getBytes("UTF-8"));
            messageDigest.update(str2.getBytes("UTF-8"));
            return bytesToHex(messageDigest.digest());
        } catch (UnsupportedEncodingException | NoSuchAlgorithmException unused) {
            return "";
        }
    }


    public static void main(String[] args) {

          String fileName = "paintings.txt";

        try (BufferedReader br = new BufferedReader(new FileReader(fileName))) {
            String line;
            int i =0;
            while ((line = br.readLine()) != null) {
                // Splitting the line by space
                String[] parts = line.split("\\s+");
                
                if (parts.length >= 2) { // Ensure there are at least two parts
                    String str = parts[0];
                    String str2 = parts[1];
                    i++;
                    if(computeHash(str, str2).equals("6E3E25FC3EBEE6CDF0B383683B261045D3DD52D5D3C106BD5F11FBCAD01C8285")){
                        System.out.println(str+" " + str2);
                    }

                }
            //System.out.println(i);

            }

        
        } catch (IOException e) {
            e.printStackTrace();
        }
        //System.out.println(getFlag2());
    }

   
}

```


<mark style="background: #BBFABBA6;">So we can compile and run this java file and we obtain this result:</mark>
`CrowdedFilmMakersDisappointWhatever SustainableLeftsMaintainOriginally`

We search for both of them in JADX and we notice that the name of the painting is `SustainableLeftsMaintainOriginally`
- ![[Pasted image 20240322180552.png]]


## Flag generation
Now we have all the information to have access to the flag.

We know that the flag is generated by the painting `SustainableLeftsMaintainOriginally` so if we take a look at the source code his java class we can see that there is the method getFlag2():
```Java
  
    public static String getFlag2() {  
        String str;  
        try {  
            MessageDigest messageDigest = MessageDigest.getInstance("SHA-256");  
            messageDigest.update("yesthisone".getBytes("UTF-8"));  
            messageDigest.update(name.getBytes("UTF-8"));  
            messageDigest.update(secret.getBytes("UTF-8"));  
            str = GalleryContent.bytesToHex(messageDigest.digest());  
        } catch (UnsupportedEncodingException | NoSuchAlgorithmException unused) {  
            Log.e(MainActivity.TAG, "Yeah this should not happen");  
            str = "";  
        }  
        return E.b(str, 0, 12, new StringBuilder("FLG_PT2{random_flag_is_"), "}");  
    }
```

But we can notice that the flag is not showed to the user:
```java
return E.b(str, 0, 12, new StringBuilder("FLG_PT2{random_flag_is_"), "}"); 
```
- example (not the right painting):
	- ![[Pasted image 20240322182322.png]]


So what we can do here is to extract the method getFlag2() and compute by ourself the flag:
```JAVA
public static String getFlag2() {

        // this is written after getting the correct painting
        String found_name = "SustainableLeftsMaintainOriginally";
        String found_secret = "CrowdedFilmMakersDisappointWhatever";
        String str;
        try {
            MessageDigest messageDigest = MessageDigest.getInstance("SHA-256");
            messageDigest.update("yesthisone".getBytes("UTF-8"));
            messageDigest.update(found_name.getBytes("UTF-8"));
            messageDigest.update(found_secret.getBytes("UTF-8"));
            str = bytesToHex(messageDigest.digest());
            

            
        } catch (UnsupportedEncodingException | NoSuchAlgorithmException unused) {
            str = "";
        }
        

        // I construct it in this way because the flag must be of the form: FLG_PT2{random_flag_is_**12 chars returned by the digest in this method**}
        return "FLG_PT2{random_flag_is_" +str.toLowerCase().substring(0,12)+ "}";
    }
```

> Note that the flag is returned in this format because is the format requested by the challenge, so we take only the first 12 chars of the generated digest, in lowercase




The flag is: `FLG_PT2{random_flag_is_0f5943ffc0fb}`

















































ho visto che i metodi sono diversi nel getID 
alcuni sono: 
```java
public static String getID() {  
        return GalleryContent.computeHash(secret, name);  
    }
    
```

altri
```java
public static String getID() {  
        return GalleryContent.computeHash(name, secret);  
    }
    
```






2. vedo nel metodo:
```java
package wien.secpriv.challenges.braf;  
  
import android.os.Bundle;  
import android.util.Log;  
import android.view.View;  
import android.widget.EditText;  
import android.widget.Toast;  
import androidx.appcompat.app.AbstractActivityC0027n;  
import java.lang.reflect.InvocationTargetException;  
  
/* loaded from: classes.dex */  
public class GuessingActivity extends AbstractActivityC0027n {  
    private boolean verify(String str) {  
        String str2;  
        Log.d(MainActivity.TAG, "got guess: " + str);  
        Class galleryContentByName = GalleryActivity.getGalleryContentByName(str);  
        if (galleryContentByName == null) {  
            return false;  
        }  
        try {  
            str2 = (String) galleryContentByName.getMethod("getID", new Class[0]).invoke(null, new Object[0]);  
        } catch (IllegalAccessException | NoSuchMethodException | InvocationTargetException unused) {  
            Log.e(MainActivity.TAG, "Yeah this should not happen");  
            str2 = "";  
        }  
        return str2.equals("6E3E25FC3EBEE6CDF0B383683B261045D3DD52D5D3C106BD5F11FBCAD01C8285");  
    }  
  
    public void checkGuess(View view) {  
        Toast.makeText(this, verify(((EditText) findViewById(R.id.input_guess)).getText().toString()) ? "Correct!" : "That's not it!", 1).show();  
    }  
  
    /* JADX INFO: Access modifiers changed from: protected */  
    @Override // androidx.fragment.app.j, androidx.activity.g, androidx.core.app.e, android.app.Activity  
    public void onCreate(Bundle bundle) {  
        super.onCreate(bundle);  
        setContentView(R.layout.activity_guessing);  
    }  
}
```
- str2.equals("6E3E25FC3EBEE6CDF0B383683B261045D3DD52D5D3C106BD5F11FBCAD01C8285"); 

provo a modificare il file smali di GuessingActivity:
![[Pasted image 20240321214123.png]]

ma non va, non mi fa ribuildare 
![[Pasted image 20240321214143.png]]


MA COMUNQUE NON MI DICE  NULLA







Nessuna immagine triggera questa exception:
![[Pasted image 20240322100709.png]]


Non so più che fare



apro le immagini e vedo se c'è qualcosa di interesante (magari un commento ) siccome sono jpg


provo anche a recuperare una flag con i metodi:
```java
  

import java.io.UnsupportedEncodingException;

import java.security.MessageDigest;

import java.security.NoSuchAlgorithmException;

  
  

/* loaded from: classes.dex */

public class DigestCreator {

public static final String image = "painting120";

public static final String name = "GiantMerciesThriveSouth";

protected static String secret = "IncredibleReturnsDiscardAllTheTime";

  

public static String getFlag2() {

  

String str;

try {

MessageDigest messageDigest = MessageDigest.getInstance("SHA-256");

messageDigest.update("yesthisone".getBytes("UTF-8"));

messageDigest.update(name.getBytes("UTF-8"));

messageDigest.update(secret.getBytes("UTF-8"));

str = bytesToHex(messageDigest.digest());

} catch (UnsupportedEncodingException | NoSuchAlgorithmException unused) {

str = "";

}

return str;

}

  
  

public static String bytesToHex(byte[] bArr) {

char[] cArr = new char[bArr.length * 2];

char[] HEX_ARRAY = "0123456789ABCDEF".toCharArray();

for (int i2 = 0; i2 < bArr.length; i2++) {

int i3 = bArr[i2] & 255;

int i4 = i2 * 2;

char[] cArr2 = HEX_ARRAY;

cArr[i4] = cArr2[i3 >>> 4];

cArr[i4 + 1] = cArr2[i3 & 15];

}

return new String(cArr);

}

  

public static void main(String[] args) {

String flag2 = DigestCreator.getFlag2();

System.out.println(flag2);

}

  

}
```
- ma è un hash di sha256, non può essere la flag