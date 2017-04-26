一个视频播放的activity，可以直接拿过来用，比较简单，直接贴个代码留作记录，效果如下：

![](http://img.blog.csdn.net/20131121171540296?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdHRkZXZz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)  

``` java
/**
 * 视频播放的activity<br>
 * 使用方法：<br>
 * 1、AndroidManifest.xml中添加此activity<br>
 * 2、Intent中存入待播放视频的URL，支持本地和网络<br>
 * 3、启动这个activity<br>
 * <br>
 * example：<br>
 * String url = "play url";<br>
 * Intent intent = new Intent();<br>
 * intent.setClass(this, VideoPlayActivity.class);<br>
 * intent.putExtra(VideoPlayActivity.KEY_URL, url);<br>
 * startActivity(intent);<br>
 *
 * @author ttworking
 */
public class VideoPlayActivity extends Activity implements OnPreparedListener, OnErrorListener {
    public static final String KEY_URL = "view_url";

    private static final String TAG = "VideoPlayActivity";
    private static final int PLAY_RETURN = 2 * 1000; // 2 seconds
    private static final String KEY_PLAY_POSITON = "paly_position";
    private static final String TOAST_ERROR_URL = "Paly url is null, please check parameter:" + KEY_URL;
    private static final String TOAST_ERROR_PLAY = "Paly error, please check url exist!";
    private static final String DIALOG_TITILE = "奋力加载中，请稍后...";

    private static String url;

    private ProgressDialog progressDialog;
    private MediaController mc;
    private VideoView videoView;
    private LinearLayout llMain;
    private LayoutParams params = new LayoutParams(LayoutParams.MATCH_PARENT, LayoutParams.MATCH_PARENT);

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        url = getIntent().getStringExtra(KEY_URL);
        if (url == null && savedInstanceState != null) {
            url = savedInstanceState.getString(KEY_URL);
        }

        if (url == null) {
            Toast.makeText(getApplicationContext(), TOAST_ERROR_URL, Toast.LENGTH_LONG).show();
            finish();
            return;
        }

        setTheme(android.R.style.Theme_NoTitleBar_Fullscreen);

        videoView = new VideoView(this);
        videoView.setVideoURI(Uri.parse(url));
        videoView.requestFocus();
        videoView.setOnPreparedListener(this);
        videoView.setOnErrorListener(this);

        mc = new MediaController(this);
        mc.setAnchorView(videoView);
        mc.setKeepScreenOn(true);

        videoView.setMediaController(mc);

        llMain = new LinearLayout(this);
        llMain.setGravity(Gravity.CENTER_VERTICAL);
        llMain.setOrientation(LinearLayout.VERTICAL);
        llMain.setLayoutParams(params);

        llMain.addView(videoView, params);
        setContentView(llMain);

        initDialog();
    }

    @Override
    protected void onSaveInstanceState(Bundle outState) {
        super.onSaveInstanceState(outState);

        int palyPosition = videoView.getCurrentPosition();
        if (palyPosition > PLAY_RETURN) {
            palyPosition -= PLAY_RETURN;
        }
        outState.putInt(KEY_PLAY_POSITON, palyPosition);
        outState.putString(KEY_URL, url);
    }

    @Override
    protected void onRestoreInstanceState(Bundle savedInstanceState) {
        super.onRestoreInstanceState(savedInstanceState);

        videoView.seekTo(savedInstanceState.getInt(KEY_PLAY_POSITON));
    }

    @Override
    public void onPrepared(MediaPlayer mp) {
        progressDialog.cancel();

        videoView.start();
    }

    @Override
    public boolean onError(MediaPlayer mp, int what, int extra) {
        Log.d(TAG, "onError:" + url);

        Toast.makeText(getApplicationContext(), TOAST_ERROR_PLAY + "\n" + url, Toast.LENGTH_LONG).show();
        progressDialog.cancel();
        finish();

        return true;
    }

    private void initDialog() {
        progressDialog = new ProgressDialog(this);
        progressDialog.setProgressStyle(ProgressDialog.STYLE_SPINNER);
        progressDialog.setMessage(DIALOG_TITILE);
        progressDialog.setCancelable(false);
        progressDialog.show();
    }
}
```

![Create by ttdevs](https://raw.githubusercontent.com/ttdevs/ttdevs.github.io/common/images/logo.png)

