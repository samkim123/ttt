## 리워드 전면비디오(Reward Interstitial Video) 광고 시작하기

|                                                        리워드 비디오 예시                                                         | EndCard 예시
|:-------------------------------------------------------------------------------------------------------------------------:|:---:|
| <img src="https://github.com/samkim123/ttt/blob/master/reward_endcar.jpg"  width="40%" height="40%"/> |<img src="https://github.com/samkim123/ttt/blob/master/reward.jpg"  width="40%" height="40%"/>


## 1. Reward Interstitial Video (리워드 전면비디오) 추가 예제
- Reward Interstitial Video 광고 뷰는 2가지 방법으로 사용 할 수 있습니다.     
  (1) 리워드 전면비디오 광고 요청 후 받은 즉시 노출   
  (2) 리워드 전면비디오 광고 요청 후 받은 뒤, 원하는 시점에 노출
- 리워드 전면비디오 광고 요청 후 받은 뒤, 원하는 시점에 노출을 시   
  (1) rewardInterstitialVideoAd.loadRewardVideoAd(); 호출   
  (2) onReceivedAd 이벤트를 받은 뒤, 원하는 시점에 if(rewardInterstitialVideoAd.hasInterstitial) // 리워드 전면비디오 광고를 성공적으로 받았는지 판단.   
  (3) rewardInterstitialVideoAd.showRewardVideoAd(); // 리워드 전면비디오 광고 노출    
 - 유의사항
   - 광고 로딩이 성공한 이후 노출하지 않고 지나치게 많은 시간이 지나가면 showRewardVideoAd() 을 호출 했을 때, 제대로 광고가 표시되지 않을 수 있습니다.
   - loadRewardVideoAd() 을 호출하고 일정 시간이 지나면 광고가 노출되어도 유효 노출로 처리되지 않을 수 있습니다.

- 아래 코드 및 샘플 프로젝트를 참조하여 원하는 방법으로 광고를 추가 해 주세요
```java
public class RewardInterstitialVideoActivity extends AppCompatActivity {

    private Button btnInterstitialVideoShow;
    private RewardInterstitialVideoAd rewardInterstitialVideoAd;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_interstitialvideo);

        btnInterstitialVideoShow = findViewById(R.id.btn_interstitial_video_show);
        
        Map<String, String> params = new HashMap<>();
        params.put("use_id", "nas");
        params.put("name", "choi");
        params.put("phone", "010-1111-1111");
		
        AdInfo adInfo = new AdInfo.Builder(Application.adUnitId_interstitialVideo) // AdMixer 플랫폼에서 발급받은 리워드 전면비디오 ADUNIT_ID
                .interstitialTimeout(0) // 초단위로 리워드 전면비디오 광고 타임아웃 설정 (기본값 : 0, 0 이면 서버지정 시간으로 처리, 서버지정 시간 : 20s)
                .maxRetryCountInSlot(-1) // 리로드 시간 내에 반복 횟수(-1 : 무한, 0 : 반복 없음, n : n번 반복)
                .isRetry(true) // 광고 재요청 설정 (true - 기본값), false 시, 1회 요청 후 바로 Callback
                .setCustomParams(params) // Reward Callback 커스텀데이터 Map형태로 추가 (선택사항) 
                .build();

        rewardInterstitialVideoAd = new RewardInterstitialVideoAd(this);
        // 이 때 설정하신 Reward Interstitial Video 의 부모 activity 는 원활한 광고 제공을 위해 hardwareAccelerated 가 true 설정되오니 참고 부탁드립니다.
        rewardInterstitialVideoAd.setAdInfo(adInfo, this);
        rewardInterstitialVideoAd.setListener(new AdListener() {
            @Override
            public void onReceivedAd(Object adView) {
                // 광고 수신 성공
                Toast.makeText(getApplicationContext(), "onReceivedAd", Toast.LENGTH_SHORT).show();
            }

            @Override
            public void onFailedToReceiveAd(Object adView, int errorCode, String errorMsg) {
                // 광고 수신 실패
                Toast.makeText(getApplicationContext(), errorMsg, Toast.LENGTH_SHORT).show();
            }

            @Override
            public void onEventAd(Object adView, AdEvent adEvent) {
                // 기타 이벤트
                switch (adEvent) {
                    case CLOSE:   // 광고 창이 닫혔을 때
                    case SKIPPED: // 동영상 SKIP 버튼 클릭 시
                        rewardInterstitialVideoAd.closeRewardVideoAd();
                        break;
                    case COMPLETION: // 동영상 재생 완료 시(리워드 지급 처리 해주세요)
                    case CLICK: // 더보기 버튼 클릭 시
                        break;
                }
            }
        });

        // 광고 요청 후 광고를 받은 뒤, 원하는 시점에 노출
        btnInterstitialVideoShow.setOnClickListener(v -> {
            if (rewardInterstitialVideoAd.hasInterstitial) {
                rewardInterstitialVideoAd.showRewardVideoAd(); // 광고를 노출한다.
            } else {
                rewardInterstitialVideoAd.loadRewardVideoAd(); // 광고를 미리 로드한다.
            }
        });
    }

    // 생명주기에 따라 아래 설정이 반드시 필요합니다.
    @Override
    protected void onDestroy() {
        if (rewardInterstitialVideoAd != null) {
            rewardInterstitialVideoAd.stopRewardVideoAd();
            rewardInterstitialVideoAd = null;
        }
        super.onDestroy();
    }
}
```

## Reward 처리
- AdListener의 onEventAd에서 동영상 재생 완료시(COMPLETION) 유저에게 리워드 지급을 처리해 주세요.
```java
     @Override
            public void onEventAd(Object adView, AdEvent adEvent) {
                // 기타 이벤트
                switch (adEvent) {
                    ...
                    ...
                    ...
                    case COMPLETION: // 동영상 재생 완료 시(리워드 지급 처리)
                    ...
                        break;
                }
            }
```

## S2S Reward Callback[선택사항]
- 유저가 리워드 광고 시청을 완료하면, 매체사가 정의한 외부 서버로 해당 유저가 리워드 광고 시청을 완료했음을 전달하는 기능입니다.
  애드믹서는 광고 시청 완료 후 바로 콜백을 수행하지만, 몇 분 정도 지연될 수 있습니다.
- 해당 기능은 파트너 사이트에서 리워드 광고 애드유닛에 대한 상세 설정을 통해 사용할 수 있습니다.

### 설정 1 : 파트너 사이트에서 콜백 서버 url 입력
- **파트너 사이트** → **미디어 관리** → **광고 설정** 에서 매체사의 콜백 서버 URL을 입력하세요.
- 유저가 해당 애드유닛을 통해 송출된 리워드 광고 시청을 완료하게 되면, 입력된 콜백 서버 URL로 리워드 콜백 데이터를 전송합니다.

|                                                              파트너 사이트                                                               |
|:----------------------------------------------------------------------------------------------------------------------------------:|
| <img src="https://github.com/samkim123/ttt/blob/master/settings.png"  width="60%" height="60%"/> |


### 기본 파라미터


|파라미터| 설명                             |예시|
|------|--------------------------------|---|
|media_key| 매체 키(애드믹서 파트너사이트에서 발급)         |12345678|
|adunit_id| 애드유닛 아이디(애드믹서 파트너사이트에서 발급)     |87654321|
|adid| Android : google Advertise ID iOS : idfa |860635ea-65bc-eaed-d355-1b5283b30b94|
|user_id| 유저 식별값 ||
|complete| 광고 시청 완료 이벤트 ||
|timestamp| complete 이벤트가 발생한 시간 |1546300800|
###### 다음 표의 파라미터가 콜백 URL로 전송됩니다.(아래 예제 URL 참조)

| {매체사콜백url}?media_key={mediakey}&adunit_id={adunitid}&adid={adid}&user_id={user_id}&complete={complete}&timestamp={timestamp} |
|:-----------------------------------------------------------------------------------------------------------------------------|


### 설정 2 : SDK CustomData 추가
- **SDK**의 CustomData 추가를 통해 콜백에서 추가 데이터를 수집할 수 있습니다.
- CustomData는 String Map 형태로 추가 해야합니다.
- 아래 코드 샘플을 참고해주세요.
```java
   //예시
    Map<String, String> params = new HashMap<>();
        params.put("useid", "nas");
        params.put("name", "choi");
        params.put("phone", "010-1111-1111");	
	
	AdInfo adInfo = new AdInfo.Builder(Application.adUnitId)
                ...
                ...
                .setCustomParams(params) // Rewarded Callbacks CustomData는 String Map형태로 추가 
                .build();
		
```

###### SDK 내 CustomData 추가에서 세팅한 Custom파라미터가 콜백 URL에 포함되어 전송됩니다. (아래 예제 URL 참조)


| {매체사콜백url}?media_key={mediakey}&adunit_id={adunitid}&adid={adid}&user_id={user_id}&complete={complete}&timestamp={timestamp}&useid=nas&name=choi&phone=010-1111-1111 |
|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------|
