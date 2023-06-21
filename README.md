## 동영상(Video) 광고 시작하기

|                                                 비디오 예시                                                 | 전면비디오 예시
|:------------------------------------------------------------------------------------------------------:|:---:|
| <img src="https://github.com/samkim123/ttt/blob/master/reward_endcard.jpg"  width="40%" height="40%"/> |<img src="https://github.com/samkim123/ttt/blob/master/video_inters.jpg"  width="40%" height="40%"/>

## 1. Video 광고 추가 예제
- 아래 코드는 Video 광고 받은 뒤, VideoAdView를 RelativeLayout에 추가한(addView) 예제 입니다.
- 배너 광고 요청 후 받은 뒤, 원하는 시점에 노출   
   (1) videoAdView.loadAd(); 로드요청   
   (2) container.removeView(videoAdView); // 기존에 있으면 배너 광고 뷰가 있으면 제거   
   (3) container.addView(videoAdView, params);   // 레이아웃에 배너 광고 뷰를 추가   
   (4) video 광고 노출
 - 유의사항
   - 광고 로딩이 성공한 이후 노출하지 않고 지나치게 많은 시간이 지나가면 addView() 을 호출 했을 때, 제대로 광고가 표시되지 않을 수 있습니다.
   - loadAd() 을 호출하고 일정 시간이 지나면 광고가 노출되어도 유효 노출로 처리되지 않을 수 있습니다.

- 아래 코드 및 샘플 프로젝트를 참조하여 원하는 방법으로 광고를 추가하십시오.
```xml
[activity_video.xml]
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="#f2a8ca">

    <RelativeLayout
        android:id="@+id/container_video"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="#000000"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent">

        <TextView
            android:id="@+id/tv_complete"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_centerInParent="true"
            android:gravity="center"
            android:text="Video Play Complete"
            android:textColor="#FFFFFF"
            android:visibility="gone" />

    </RelativeLayout>


</androidx.constraintlayout.widget.ConstraintLayout>
```

```java
public class VideoActivity extends AppCompatActivity {

    private VideoAdView videoAdView;
    private RelativeLayout container;
    private TextView tvComplete;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_video);

        container = findViewById(R.id.container_video);
        tvComplete = findViewById(R.id.tv_complete);

        AdInfo adInfo = new AdInfo.Builder(Application.adUnitId_video) // AdMixer 플랫폼에서 발급받은 배너 ADUNIT_ID
                .isRetry(true) // 광고 재요청 설정 (true - 기본값), false 시, 1회 요청 후 바로 Callback
                .build();

        videoAdView = new VideoAdView(this);
        // 이 때 설정하신 video 의 부모 activity 는 원활한 광고 제공을 위해 hardwareAccelerated 가 true 설정되오니 참고 부탁드립니다.
        videoAdView.setAdInfo(adInfo, this);
        videoAdView.setAdViewListener(new AdListener() {
            @Override
            public void onReceivedAd(Object o) {
                // 광고 수신 성공
                Toast.makeText(VideoActivity.this, "onReceivedAd", Toast.LENGTH_SHORT).show();
                // 광고를 노출한다.
                RelativeLayout.LayoutParams params = new RelativeLayout.LayoutParams(ViewGroup.LayoutParams.MATCH_PARENT, ViewGroup.LayoutParams.WRAP_CONTENT);
                params.addRule(RelativeLayout.CENTER_IN_PARENT);
                container.removeView(videoAdView);
                container.addView(videoAdView, params); // 레이아웃에 Video 광고 뷰를 추가 [ 광고 노출 ]
            }

            @Override
            public void onFailedToReceiveAd(Object o, int i, String s) {
                // 광고 수신 실패
                Toast.makeText(VideoActivity.this, s, Toast.LENGTH_SHORT).show();
            }

            @Override
            public void onEventAd(Object o, AdEvent adEvent) {
                // 기타 이벤트
                Toast.makeText(VideoActivity.this, adEvent.toString(), Toast.LENGTH_SHORT).show();
                switch (adEvent) {
                    case COMPLETION: // 동영상 재생 완료 시
                    case SKIPPED:    // 동영상 SKIP 버튼 클릭 시
                        RelativeLayout.LayoutParams params = new RelativeLayout.LayoutParams(videoAdView.getWidth(), videoAdView.getHeight());
                        params.addRule(RelativeLayout.CENTER_IN_PARENT);
                        tvComplete.setLayoutParams(params);
                        tvComplete.setVisibility(View.VISIBLE);
                        break;
                    case CLICK: // 더보기 버튼 클릭 시
                        break;
                }
            }
        });

        // 광고 요청 후 광고를 받은 뒤, 원하는 시점에 노출
        videoAdView.loadAd(); // 광고를 미리 로드한다.
    }

    // 생명주기에 따라 아래 설정이 반드시 필요합니다.
    @Override
    protected void onResume() {
        if (videoAdView != null) {
            videoAdView.onResume();
        }
        super.onResume();
    }

    @Override
    protected void onPause() {
        if (videoAdView != null) {
            videoAdView.onPause();
        }

        super.onPause();
    }

    @Override
    protected void onDestroy() {
        if (videoAdView != null) {
            videoAdView.onDestroy();
            videoAdView = null;
        }

        super.onDestroy();
    }
}
```

## 2. Interstitial Video (전면 비디오 광고) 추가 예제
- Interstitial Video 광고 뷰는 아래와 같이 사용 할 수 있습니다.
- 전면 비디오 광고 요청 후 받은 뒤, 원하는 시점에 노출을 원할 시   
  (1) interstitialVideoAdView.loadInterstitialVideoAd(); 후    
  (2) onReceivedAd 이벤트를 받은 뒤, 원하는 시점에 if(interstitialVideoAdView.hasInterstitial) // 전면 비디오 광고를 성공적으로 받았는지 판단.   
  (3) interstitialVideoAdView.showInterstitialVideoAd(); // 전면 비디오 광고 노출    
 - 유의사항
   - 광고 로딩이 성공한 이후 노출하지 않고 지나치게 많은 시간이 지나가면 showInterstitialVideoAd() 을 호출 했을 때, 제대로 광고가 표시되지 않을 수 있습니다.
   - loadInterstitialVideoAd() 을 호출하고 일정 시간이 지나면 광고가 노출되어도 유효 노출로 처리되지 않을 수 있습니다.

- 아래 코드 및 샘플 프로젝트를 참조하여 원하는 방법으로 광고를 추가하십시오.
```java
public class InterstitialVideoActivity extends AppCompatActivity {

    private Button btnInterstitialVideoShow;
    private ProgressBar progressBar;
    private InterstitialVideoAd interstitialVideoAdView;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_interstitialvideo);

        progressBar = findViewById(R.id.loading_bar);
        btnInterstitialVideoShow = findViewById(R.id.btn_interstitial_video_show);

        AdInfo adInfo = new AdInfo.Builder(Application.adUnitId_interstitialVideo) // AdMixer 플랫폼에서 발급받은 전면 비디오 ADUNIT_ID
                .interstitialTimeout(0) // 초단위로 전면 광고 타임아웃 설정 (기본값 : 0, 0 이면 서버지정 시간으로 처리, 서버지정 시간 : 20s)
                .maxRetryCountInSlot(-1) // 리로드 시간 내에 반복 횟수(-1 : 무한, 0 : 반복 없음, n : n번 반복)
                .isRetry(true) // 광고 재요청 설정 (true - 기본값), false 시, 1회 요청 후 바로 Callback
                .build();

        interstitialVideoAdView = new InterstitialVideoAd(this);
        // 이 때 설정하신 Interstitial Video 의 부모 activity 는 원활한 광고 제공을 위해 hardwareAccelerated 가 true 설정되오니 참고 부탁드립니다.
        interstitialVideoAdView.setAdInfo(adInfo, this);
        interstitialVideoAdView.setListener(new AdListener() {
            @Override
            public void onReceivedAd(Object o) {
                // 광고 수신 성공
                Toast.makeText(InterstitialVideoActivity.this, "onReceivedAd", Toast.LENGTH_SHORT).show();
                btnInterstitialVideoShow.setText("전면 비디오 광고보기");
                progressBar.setVisibility(View.GONE);
            }

            @Override
            public void onFailedToReceiveAd(Object o, int i, String s) {
                // 광고 수신 실패
                Toast.makeText(InterstitialVideoActivity.this, s, Toast.LENGTH_SHORT).show();
            }

            @Override
            public void onEventAd(Object o, AdEvent adEvent) {
                // 기타 이벤트
                switch (adEvent) {
                    case CLOSE:   // 광고 창이 닫혔을 때
                    case SKIPPED: // 동영상 SKIP 버튼 클릭 시
                        interstitialVideoAdView.closeInterstitialVideoAd();
                        btnInterstitialVideoShow.setText("전면 비디오 광고요청");
                        break;
                    case COMPLETION: // 동영상 재생 완료 시
                    case CLICK: // 더보기 버튼 클릭 시
                        break;
                }
            }
        });

        // 광고 요청 후 광고를 받은 뒤, 원하는 시점에 노출
        btnInterstitialVideoShow.setOnClickListener(v -> {
            if (interstitialVideoAdView.hasInterstitial)
                interstitialVideoAdView.showInterstitialVideoAd(); // 광고를 노출한다.
            else {

                interstitialVideoAdView.loadInterstitialVideoAd(); // 광고를 미리 로드한다.
                progressBar.setVisibility(View.VISIBLE);
            }
        });
    }

    // 생명주기에 따라 아래 설정이 반드시 필요합니다.
    @Override
    protected void onDestroy() {
        if (interstitialVideoAdView != null) {
            interstitialVideoAdView.stopInterstitialVideoAd();
            interstitialVideoAdView = null;
        }
        super.onDestroy();
    }
}
```
