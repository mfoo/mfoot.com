---
comments: true
date: 2011-07-17 09:22:08
layout: post
slug: sending-photos-to-a-python-web-server-with-android
title: Sending photos to a Python web server with Android
wordpress_id: 196
categories:
- Android
- Programming
tags:
- android
- base64
- java
- Photography
- python
---

I recently involved in a project where we performed some computer vision object recognition and classification using neural networks and SVMs. It was necessary to use photos taken from an Android phone and it took a little while to get it working. Below is the solution I used, which involved creating a tiny Android application that took a photo, Base64 encoded it, then sent the data over HTTP POST to a Python BaseHTTPRequestHandler subclass which decoded the file and wrote it to disk.



The solution uses a slightly modified version of [this example for Android](http://coderzheaven.com/2011/04/25/android-upload-an-image-to-a-server/) that uses [this Java Base64 encode/decode class](http://iharder.sourceforge.net/current/java/base64/). The Python script is based on [an example by Jon Berg](http://fragments.turtlemeat.com/pythonwebserver.php).

The Python web server is below:

[sourcecode language="python"]
#!/usr/bin/env python

import cgi
from BaseHTTPServer import BaseHTTPRequestHandler, HTTPServer

class MyHandler(BaseHTTPRequestHandler):

    def do_GET(self):
        print "Somebody made a POST request."
        return

    def do_POST(self):
        """
        Take the Base64 encoded string that the phone transmit, decode it and
        save it as an image file called phone.jpg.
        """
        global rootnode
        try:
            ctype, pdict = cgi.parse_header(self.headers.getheader('content-type'))
            print ctype
            print pdict
            if ctype == 'multipart/form-data':
                query=cgi.parse_multipart(self.rfile, pdict)
            elif ctype == 'application/x-www-form-urlencoded':
                length = int(self.headers.getheader('content-length'))
                query = cgi.parse_qs(self.rfile.read(length), keep_blank_values=1)

            self.send_response(301)
            self.send_header('Content-type', 'text/html')
            self.end_headers()

            upfilecontent = query.get('upfile')
            self.wfile.write("POST OK.

");
            f = open("phone.jpg", "wb")
            f.write(upfilecontent[0].decode('base64'))
            f.close()
            print("File received.")

            return
        except Exception, err:
            print Exception, err
            pass

def main():
    try:
        server = HTTPServer(('', 8000), MyHandler)
        print 'started httpserver...'
        server.serve_forever()
    except KeyboardInterrupt:
        print '^C received, shutting down server'
        server.socket.close()

if __name__ == '__main__':
    main()
[/sourcecode]

And here's the Android Activity (remember to add the [CAMERA permission](http://developer.android.com/reference/android/Manifest.permission.html#CAMERA) to your manifest and add the above linked Base64 encoding class to your workspace and package).

[sourcecode language="java"]
package com.elec6024.eardetection;

/**
 * This Android Activity takes a photo and uploads it to a server.
 * The server upload HTTP POST code is modified from this:
 * http://coderzheaven.com/index.php/2011/04/android-upload-an-image-to-a-server/
 */
import java.io.ByteArrayOutputStream;
import java.io.File;
import java.io.IOException;
import java.io.InputStream;
import java.util.ArrayList;

import org.apache.http.HttpResponse;
import org.apache.http.NameValuePair;
import org.apache.http.client.HttpClient;
import org.apache.http.client.entity.UrlEncodedFormEntity;
import org.apache.http.client.methods.HttpPost;
import org.apache.http.impl.client.DefaultHttpClient;
import org.apache.http.message.BasicNameValuePair;

import android.app.Activity;
import android.content.Intent;
import android.graphics.Bitmap;
import android.graphics.BitmapFactory;
import android.net.Uri;
import android.os.Bundle;
import android.os.Environment;
import android.provider.MediaStore;
import android.view.View;
import android.view.View.OnClickListener;
import android.widget.Button;
import android.widget.Toast;

public class MainActivity extends Activity
{
	private Uri outputFileUri;
	private InputStream inputStream;
	public static final int PICTURE_ACTIVITY = 35434;

    /* Override the onCreate method */
    @Override
    public void onCreate(Bundle savedInstanceState)
    {
        super.onCreate(savedInstanceState);

        setContentView(R.layout.main);
		final Button cameraButton = (Button)findViewById(R.id.camera_button);
		cameraButton.setOnClickListener(new OnClickListener() {
			@Override
			public void onClick(View v){
				// Check if there's external storage available and that it's writeable.
				boolean mExternalStorageAvailable = false;
				boolean mExternalStorageWriteable = false;
				String state = Environment.getExternalStorageState();

				if (Environment.MEDIA_MOUNTED.equals(state)) {
				    // We can read and write the media
				    mExternalStorageAvailable = mExternalStorageWriteable = true;
				} else if (Environment.MEDIA_MOUNTED_READ_ONLY.equals(state)) {
				    // We can only read the media
				    mExternalStorageAvailable = true;
				    mExternalStorageWriteable = false;
				} else {
				    // Something else is wrong. It may be one of many other states, but all we need
				    //  to know is we can neither read nor write
				    mExternalStorageAvailable = mExternalStorageWriteable = false;
				}

				if(mExternalStorageAvailable && mExternalStorageWriteable){
					Intent cameraIntent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE); // Normally you would populate this with your custom intent.
					File file = new File(Environment.getExternalStorageDirectory(),"imagefile.jpg");
					outputFileUri = Uri.fromFile(file);
					cameraIntent.putExtra(MediaStore.EXTRA_OUTPUT, outputFileUri);
					startActivityForResult(cameraIntent, PICTURE_ACTIVITY);
				}
			}
		});
    }

	@Override
	protected void onActivityResult(int requestCode, int resultCode, Intent intent) {
		super.onActivityResult(requestCode, resultCode, intent);

		if (requestCode == PICTURE_ACTIVITY && resultCode == Activity.RESULT_OK) {

			File file = new File(Environment.getExternalStorageDirectory(),"imagefile.jpg");
			if(file.exists()){
				ByteArrayOutputStream stream = new ByteArrayOutputStream();

				// Enclose this in a scope so that when it's over we can call the garbage collector (the phone doesn't have a lot of memory!)
				{
					Bitmap image = BitmapFactory.decodeFile(file.getPath());
					Bitmap scaled = Bitmap.createScaledBitmap(image, (int)(image.getWidth() * 0.3), (int)(image.getHeight() * 0.3), false);
					scaled.compress(Bitmap.CompressFormat.JPEG, 90, stream);
				}

				System.gc();

			    byte [] byte_arr = stream.toByteArray();
			    String image_str = Base64.encodeBytes(byte_arr);
			    ArrayList nameValuePairs = new ArrayList();

			    nameValuePairs.add(new BasicNameValuePair("upfile",image_str));

			    try {
			        HttpClient httpclient = new DefaultHttpClient();
			        final String URL = "http://192.168.2.3:8000";
			        HttpPost httppost = new HttpPost(URL);
			        httppost.setEntity(new UrlEncodedFormEntity(nameValuePairs));
			        HttpResponse response = httpclient.execute(httppost);
			        String the_string_response = convertResponseToString(response);
			        Toast.makeText(MainActivity.this, "Response " + the_string_response, Toast.LENGTH_LONG).show();
			    } catch(Exception e){
			          Toast.makeText(MainActivity
			        		  .this, "ERROR " + e.getMessage(), Toast.LENGTH_LONG).show();
			          System.out.println("Error in http connection "+e.toString());
			          e.printStackTrace();
			    }
			}
		}
	}

    public String convertResponseToString(HttpResponse response) throws IllegalStateException, IOException{
        String res = "";
        StringBuffer buffer = new StringBuffer();
        inputStream = response.getEntity().getContent();
        int contentLength = (int) response.getEntity().getContentLength(); //getting content length…..
	        Toast.makeText(MainActivity.this, "contentLength : " + contentLength, Toast.LENGTH_LONG).show();
        if (contentLength < 0){         }         else{                byte[] data = new byte[512];                int len = 0;                try                {                    while (-1 != (len = inputStream.read(data)) )                    {                        buffer.append(new String(data, 0, len)); //converting to string and appending  to stringbuffer…..                    }                }                catch (IOException e)                {                    e.printStackTrace();                }                try                {                    inputStream.close(); // closing the stream…..                }                catch (IOException e)                {                    e.printStackTrace();                }                res = buffer.toString();     // converting stringbuffer to string…..                Toast.makeText(MainActivity.this, "Result : " + res, Toast.LENGTH_LONG).show();                //System.out.println("Response => " +  EntityUtils.toString(response.getEntity()));
        }
        return res;
   }
}

[/sourcecode]

You'll also need the layout file for the application:

[sourcecode language="xml"]
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent"
    >
    <ImageView android:layout_weight="2" 
        android:id="@+id/imageview"
        android:layout_width="fill_parent"
        android:layout_height="fill_parent" />
    <Button
	android:id="@+id/camera_button"
	android:layout_weight="1"
	android:layout_width="fill_parent"
	android:layout_height="wrap_content"
	android:text="@string/camera_button_text" 
	android:textSize="@dimen/big_text"/>
</LinearLayout>


[/sourcecode]


