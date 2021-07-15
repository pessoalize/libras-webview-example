# Pessoalize

## 🔖 Sobre
O presente repositório define as especificações funcionais para a utilização do sistema Pessoalize via WebView no Android.

## 🚀 Tecnologias utilizadas ##
O projeto foi desenvolvido utilizando as seguintes tecnologias:
- [Java](https://www.java.com/pt-BR/)
- [Android](https://www.android.com/intl/pt-BR_br/)

## ❗ Requisitos
- Android SDK 21+

## 📁 Como usar ##

Insira no `AndroidManifest.xml` (se necessário):
```sh
<application
    android:requestLegacyExternalStorage="true"
    android:usesCleartextTraffic="true"...
```

Adicione as permissões no seu `AndroidManifest.xml`:
```sh
<uses-permission android:name="android.permission.INTERNET"/>
<uses-permission android:name="android.permission.CAMERA" />
<uses-permission android:name="android.permission.RECORD_AUDIO" />
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
```

e no seu código: 
```sh
ActivityCompat.requestPermissions(MainActivity.this, new String[]{
    Manifest.permission.CAMERA,
    Manifest.permission.RECORD_AUDIO,
    Manifest.permission.READ_EXTERNAL_STORAGE
}, YOUR_REQUEST_CODE);
```

Defina no seu `XML`:
```sh
<WebView
    android:id="@+id/my_webview"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
/>
```

Defina na sua `Activity`:
```sh
public class MyWebViewActivity extends AppCompatActivity {
    private WebView webView;
    private ValueCallback<Uri[]> mFilePathCallback;
    private final int REQUEST_UPLOAD_FILE = 100;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        Toolbar toolbar = findViewById(R.id.toolbar);
        setSupportActionBar(toolbar);

        webView = findViewById(R.id.my_webview);
        webView.getSettings().setJavaScriptEnabled(true);
        webView.getSettings().setDomStorageEnabled(true);
        
        webView.setWebChromeClient(new WebChromeClient(){
            @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
            @Override
            public void onPermissionRequest(final PermissionRequest request) {
                request.grant(request.getResources());
            }

            @Override
            public boolean onShowFileChooser(WebView webView, ValueCallback<Uri[]> filePathCallback, FileChooserParams fileChooserParams) {
                mFilePathCallback = filePathCallback;

                Intent i = new Intent(Intent.ACTION_GET_CONTENT);
                i.addCategory(Intent.CATEGORY_OPENABLE);
                i.setType("*/*");

                startActivityForResult(Intent.createChooser(i, "File Chooser"), REQUEST_UPLOAD_FILE);
                return true;
            }
        });
        
        webView.loadUrl("https://inclusao-staging.pessoalize.com/{{SEU_ID}}?permitandroid=true");
    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
        if(requestCode == REQUEST_UPLOAD_FILE) {
            if(mFilePathCallback == null)
                return;

            Uri result = data == null || resultCode != RESULT_OK ? null : Uri.parse(data.getDataString());

            if(result != null) {
                mFilePathCallback.onReceiveValue(new Uri[]{ result });
            } else {
                mFilePathCallback.onReceiveValue(null);
            }
        }

        super.onActivityResult(requestCode, resultCode, data);
    }
}
```

## ⚽ Exemplo
Crie um arquivo chamado `activity_main.xml`
```sh
<?xml version="1.0" encoding="utf-8"?>
<androidx.coordinatorlayout.widget.CoordinatorLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <com.google.android.material.appbar.AppBarLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:theme="@style/Theme.Pessoalize.AppBarOverlay">

        <androidx.appcompat.widget.Toolbar
            android:id="@+id/toolbar"
            android:layout_width="match_parent"
            android:layout_height="?attr/actionBarSize"
            android:background="?attr/colorPrimary"
            app:popupTheme="@style/Theme.Pessoalize.PopupOverlay" />

    </com.google.android.material.appbar.AppBarLayout>

    <WebView
        android:id="@+id/my_webview"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>

    <ProgressBar
        android:id="@+id/progress_bar"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:visibility="gone"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

</androidx.coordinatorlayout.widget.CoordinatorLayout>
```

Crie um classe chamada `MainActivity.java`
```sh
public class MainActivity extends AppCompatActivity {
    private WebView webView;
    private ProgressBar progressBar;
    private ValueCallback<Uri[]> mFilePathCallback;

    private final int REQUEST_UPLOAD_FILE = 100;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        Toolbar toolbar = findViewById(R.id.toolbar);
        setSupportActionBar(toolbar);

        ActivityCompat.requestPermissions(MainActivity.this, new String[]{
            Manifest.permission.CAMERA,
            Manifest.permission.RECORD_AUDIO,
            Manifest.permission.MODIFY_AUDIO_SETTINGS,
            Manifest.permission.READ_EXTERNAL_STORAGE
        }, 1);

        progressBar = findViewById(R.id.progress_bar);
        progressBar.setVisibility(View.VISIBLE);

        webView = findViewById(R.id.my_webview);
        webView.getSettings().setJavaScriptEnabled(true);
        webView.getSettings().setDomStorageEnabled(true);

        webView.setWebChromeClient(new WebChromeClient(){
            @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
            @Override
            public void onPermissionRequest(final PermissionRequest request) {
                request.grant(request.getResources());
            }

            @Override
            public boolean onShowFileChooser(WebView webView, ValueCallback<Uri[]> filePathCallback, FileChooserParams fileChooserParams) {
                mFilePathCallback = filePathCallback;

                Intent i = new Intent(Intent.ACTION_GET_CONTENT);
                i.addCategory(Intent.CATEGORY_OPENABLE);
                i.setType("*/*");

                startActivityForResult(Intent.createChooser(i, "File Chooser"), REQUEST_UPLOAD_FILE);

                return true;
            }
        });

        webView.setWebViewClient(new WebViewClient(){
            @Override
            public void onPageStarted(WebView view, String url, Bitmap favicon) {
                progressBar.setVisibility(View.VISIBLE);
            }

            @Override
            public void onPageFinished(WebView view, String url) {
                progressBar.setVisibility(View.GONE);
            }
        });

        webView.loadUrl("https://inclusao-staging.pessoalize.com/{{SEU_ID}}?permitandroid=true");
    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
        if(requestCode == REQUEST_UPLOAD_FILE) {
            if(mFilePathCallback == null)
                return;

            Uri result = data == null || resultCode != RESULT_OK ? null : Uri.parse(data.getDataString());

            if(result != null) {
                mFilePathCallback.onReceiveValue(new Uri[]{ result });
            } else {
                mFilePathCallback.onReceiveValue(null);
            }
        }

        super.onActivityResult(requestCode, resultCode, data);
    }
}
```
