# SoundTouchDemo
### 1. 官网下载源码
[SoundTouch Audio Processing Library](http://www.surina.net/soundtouch/)

解压

### 复制代码到`src/main/cpp/`
#### 复制 include
![image.png](https://upload-images.jianshu.io/upload_images/1877190-c283be0c18925b25.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 复制 source/SoundTouch
![image.png](https://upload-images.jianshu.io/upload_images/1877190-031b83b9a8141034.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 复制 source/SoundStretch下部分代码
因为要用到wavFile所以就复制这两个
![image.png](https://upload-images.jianshu.io/upload_images/1877190-1eb3318091deff84.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#### 示例复制
##### cpp
`source\Android-lib\jni\soundtouch-jni.cpp`
![image.png](https://upload-images.jianshu.io/upload_images/1877190-eb51dd1b17252481.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### java
将示例的java文件复制到自己的包下面
![image.png](https://upload-images.jianshu.io/upload_images/1877190-7ded0dc6ae510783.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### 修改
#### 修改源代码 `STTypes.h`
##### 修改采样率为16位
![image.png](https://upload-images.jianshu.io/upload_images/1877190-47accaea2fa0c782.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 注释优化选项（实际上不注释也能正常使用？）
![image.png](https://upload-images.jianshu.io/upload_images/1877190-fec9bdee020f3899.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 修改JNI的方法名的包名
一口气全部替换掉
![image.png](https://upload-images.jianshu.io/upload_images/1877190-407ab0bde8d8a1ca.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


##### 修改cmake
- 添加include文件
```cmake
include_directories ("src/main/cpp/include")
```
- 添加souch库
将刚刚复制的源码复制上来
```cmake
add_library( # Sets the name of the library.
             soundtouch-lib

             # Sets the library as a shared library.
             SHARED

             # Provides a relative path to your source file(s).
             src/main/cpp/soundtouch-jni.cpp
             src/main/cpp/SoundTouch/AAFilter.cpp
             src/main/cpp/SoundTouch/BPMDetect.cpp
             src/main/cpp/SoundTouch/cpu_detect_x86.cpp
             src/main/cpp/SoundTouch/FIFOSampleBuffer.cpp
             src/main/cpp/SoundTouch/FIRFilter.cpp
             src/main/cpp/SoundTouch/InterpolateCubic.cpp
             src/main/cpp/SoundTouch/InterpolateLinear.cpp
             src/main/cpp/SoundTouch/InterpolateShannon.cpp
             src/main/cpp/SoundTouch/mmx_optimized.cpp
             src/main/cpp/SoundTouch/PeakFinder.cpp
             src/main/cpp/SoundTouch/RateTransposer.cpp
             src/main/cpp/SoundTouch/SoundTouch.cpp
             src/main/cpp/SoundTouch/sse_optimized.cpp
             src/main/cpp/SoundTouch/TDStretch.cpp
             src/main/cpp/SoundStretch/WavFile.cpp
             )
```

- 链接库
这里需要单独写一个，不要和原来的native-lib放在一起
```cmake
target_link_libraries( # Specifies the target library.
                       soundtouch-lib

                       # Links the target library to the log library
                       # included in the NDK.
                       ${log-lib} )
```

- java文件中修改库的名字
![image.png](https://upload-images.jianshu.io/upload_images/1877190-2e5ba565a4b40c17.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这样就可以了~~

### 尝试处理一下`wav`文件
复制和修改实例代码中的`ExampleActivity`
- 修改成如下
```java
public class ExampleActivity extends Activity implements OnClickListener {
    TextView textViewConsole = null;
    TextView editSourceFile = null;
    TextView editOutputFile = null;
    EditText editTempo = null;
    EditText editPitch = null;
    CheckBox checkBoxPlay = null;

    StringBuilder consoleText = new StringBuilder();
    private File srcFile;


    /// Called when the activity is created
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_example);

        textViewConsole = (TextView) findViewById(R.id.textViewResult);
        editSourceFile = (TextView) findViewById(R.id.editTextSrcFileName);
        editOutputFile = (TextView) findViewById(R.id.editTextOutFileName);

        editTempo = (EditText) findViewById(R.id.editTextTempo);
        editPitch = (EditText) findViewById(R.id.editTextPitch);

        Button buttonFileSrc = (Button) findViewById(R.id.buttonSelectSrcFile);
        Button buttonProcess = (Button) findViewById(R.id.buttonProcess);
        buttonFileSrc.setOnClickListener(this);
        buttonProcess.setOnClickListener(this);

        checkBoxPlay = (CheckBox) findViewById(R.id.checkBoxPlay);

        // Check soundtouch library presence & version
        checkLibVersion();

        //获取权限
        if (ContextCompat.checkSelfPermission(this, Manifest.permission.WRITE_EXTERNAL_STORAGE) != PackageManager.PERMISSION_GRANTED) {
            ActivityCompat.requestPermissions(this, new String[]{Manifest.permission.WRITE_EXTERNAL_STORAGE}, 1);
        }
    }


    /// Function to append status text onto "console box" on the Activity
    public void appendToConsole(final String text) {
        // run on UI thread to avoid conflicts
        runOnUiThread(new Runnable() {
            public void run() {
                consoleText.append(text);
                consoleText.append("\n");
                textViewConsole.setText(consoleText);
            }
        });
    }


    /// print SoundTouch native library version onto console
    private void checkLibVersion() {
        String ver = SoundTouch.getVersionString();
        appendToConsole("SoundTouch native library version = " + ver);
    }


    /// Button click handler
    @Override
    public void onClick(View arg0) {
        switch (arg0.getId()) {
            case R.id.buttonSelectSrcFile:
                Intent intent = new Intent();
                intent.setType("audio/wav");
                intent.setAction(Intent.ACTION_GET_CONTENT);
                startActivityForResult(intent, 2);
                break;

            case R.id.buttonProcess:
                // button "process" pushed
                process();
                break;
        }

    }


    /// Play audio file
    protected void playWavFile(String fileName) {
        File file2play = new File(fileName);
        Intent i = new Intent();
        i.setAction(Intent.ACTION_VIEW);
        i.addFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION);
        Uri photoURI = FileProvider.getUriForFile(this, getApplicationContext().getPackageName() + ".my.package.name.provider", file2play);
        i.setDataAndType(photoURI, "audio/wav");
        startActivity(i);
    }


    /// Helper class that will execute the SoundTouch processing. As the processing may take
    /// some time, run it in background thread to avoid hanging of the UI.
    @SuppressLint("StaticFieldLeak")
    private class ProcessTask extends AsyncTask<ProcessTask.Parameters, Integer, Long> {
        /// Helper class to store the SoundTouch file processing parameters
        public final class Parameters {
            String inFileName;
            String outFileName;
            float tempo;
            float pitch;
        }


        /// Function that does the SoundTouch processing
        public final long doSoundTouchProcessing(Parameters params) {

            SoundTouch st = new SoundTouch();
            st.setTempo(params.tempo);
            st.setPitchSemiTones(params.pitch);
            Log.i("SoundTouch", "process file " + params.inFileName);
            long startTime = System.currentTimeMillis();
            int res = st.processFile(params.inFileName, params.outFileName);
            long endTime = System.currentTimeMillis();
            float duration = (endTime - startTime) * 0.001f;

            Log.i("SoundTouch", "process file done, duration = " + duration);
            appendToConsole("Processing done, duration " + duration + " sec.");
            if (res != 0) {
                String err = SoundTouch.getErrorString();
                appendToConsole("Failure: " + err);
                return -1L;
            }

            // Play file if so is desirable
            if (checkBoxPlay.isChecked()) {
                playWavFile(params.outFileName);
            }
            return 0L;
        }


        /// Overloaded function that get called by the system to perform the background processing
        @Override
        protected Long doInBackground(Parameters... aparams) {
            return doSoundTouchProcessing(aparams[0]);
        }

    }


    /// process a file with SoundTouch. Do the processing using a background processing
    /// task to avoid hanging of the UI
    protected void process() {
        try {
            ProcessTask task = new ProcessTask();
            String text = "/sdcard/Download/soundtouch-output" + System.currentTimeMillis() + ".wav";
            editOutputFile.setText(text);

            ProcessTask.Parameters params = task.new Parameters();
            // parse processing parameters
            params.inFileName = editSourceFile.getText().toString();
            params.outFileName = editOutputFile.getText().toString();
            params.tempo = 0.01f * Float.parseFloat(editTempo.getText().toString());
            params.pitch = Float.parseFloat(editPitch.getText().toString());

            // update UI about status
            appendToConsole("Process audio file :" + params.inFileName + " => " + params.outFileName);
            appendToConsole("Tempo = " + params.tempo);
            appendToConsole("Pitch adjust = " + params.pitch);

            Toast.makeText(this, "Starting to process file " + params.inFileName + "...", Toast.LENGTH_SHORT).show();

            // start SoundTouch processing in a background thread
            task.execute(params);
//			task.doSoundTouchProcessing(params);	// this would run processing in main thread

        } catch (Exception exp) {
            exp.printStackTrace();
        }

    }


    @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        if (requestCode == 1 && permissions.length == 1 && grantResults.length == 1 && grantResults[0] == PackageManager.PERMISSION_GRANTED) {

        } else {
            Toast.makeText(this, "no permission GG", Toast.LENGTH_SHORT).show();
            finish();
        }
    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        // 选取图片的返回值
        if (requestCode == 2) {
            if (resultCode == RESULT_OK && data != null && data.getData() != null) {
                Uri uri = data.getData();
                Cursor cursor = null;
                try {
                    String[] filePathColumn = {MediaStore.Video.Media.DATA};

                    cursor = getContentResolver().query(uri,
                            filePathColumn, null, null, null);
                    if (cursor == null) {
                        return;
                    }
                    cursor.moveToFirst();
                    String filePath = cursor.getString(0); // 图片编号
                    Toast.makeText(this, "choose file path=" + filePath, Toast.LENGTH_SHORT).show();
                    srcFile = new File(filePath);
                    editSourceFile.setText(srcFile.getAbsolutePath());
                } finally {
                    if (cursor != null) {
                        cursor.close();
                    }
                }
            }
        }
    }
}
```
- 添加FileProvider
```java
public class GenericFileProvider extends FileProvider {}
```
- Manifest 权限和provider
```xml
  <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />

  <application
  >
   ...
        <provider
            android:name=".GenericFileProvider"
            android:authorities="${applicationId}.my.package.name.provider"
            android:exported="false"
            android:grantUriPermissions="true">
            <meta-data
                android:name="android.support.FILE_PROVIDER_PATHS"
                android:resource="@xml/provider_paths"/>
        </provider>
    </application>
```

### Done
![image.png](https://upload-images.jianshu.io/upload_images/1877190-4cd66bbcb8764d17.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### demo位置


