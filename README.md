백그라운드 푸쉬 관련해서 애플 개발자 페이지에서 When a background update notification is delivered to the user’s device, iOS wakes up your app in the background and gives it up to 30 seconds to run. 라고 설명하고 있습니다.

제가 일전에 질문게시판에 올린 http://iphonedev.co.kr/iOSDevQnA/62375 을 보시면 백그라운드 푸쉬가 정상적이지 않다는 stackoverflow 과 애플 개발자 게시판에 관련 사항을 문의 하는 글들이 많았습니다.

11.1 버전부터 수정되어 정상적으로 작동하는 것으로 보여졌으나 11.2(현재 버전)까지 중에 일부 버전에서 버그가 있지 않았나 합니다. 

확인차 제가 가지고 있는 디바이스에서 테스트 한 결과 
11.3 beta 4 에서 정상적으로 작동하고 있습니다
11.2.6 에서는 앱이 백그라운드 상태에 있을 때 정상적이고, 앱이 종료된 상태에서는 백그라운드 푸쉬 처리를 못하고 있는 것 같습니다.
.(추가 앱이 종료된 상태에서는 백그라운드 푸쉬 처리를 못합니다)

백그라운드 상태,  종료된 상태  이 두 상태 모두 푸쉬를 받아서 정상적으로 처리하고 있습니다.

백그라운드(silent) 푸쉬 적용하는 방법에 대해서 정리하겠습니다.

서버 : 
{"aps":{"content-available":1}}  간단하게 이처럼 디바이스로 푸쉬를 보냅니다.
서버에서 alert 값이 들어가면 알림이 표시되고 백그라운드 푸쉬가 안되니 이점 유의 하시기 바랍니다.

디바이스 :  
Targets -> Capabilities ->  Push Notifications : ON
Targets -> Capabilities ->  Background Modes : ON (Background fetch, Remote notifications 체크)

swift 소스 :

func application(_ application: UIApplication, didReceiveRemoteNotification userInfo: [AnyHashable : Any], fetchCompletionHandler completionHandler: @escaping (UIBackgroundFetchResult) -> Void){
        
//푸쉬 받았을 때 처리부
completionHandler(.newData) 
}


푸쉬 메세지 받는 부분인데 백그라운드 도 동일합니다.

만약 뱃지 카운트 증가를 하고 싶으시다면 

UIApplication.shared.applicationIconBadgeNumber = 뱃지 수
// 저 같은 경우 푸쉬당 1 의 뱃지 수를 증가 시켜서 아래와 같이 처리
UIApplication.shared.applicationIconBadgeNumber += 1

처리부에 넣어주시면 됩니다. 
추가로, 처리부 안에서 변수 선언해서 작업하는 부분은 잘 안되는 것 같습니다.
(백그라운드 푸쉬 목적이 다운로드 용이라 그런지 ...)

혹시 제가 잘못 정리했거나 틀린 부분이 있으면 알려주시면 수정하겠습니다.

추가- 백그라운드 푸쉬를 선택하게 된 것은 뱃지 값을 앱에서 처리하고 싶었습니다.
서버에서는 메세지만 던져주면 앱에서 뱃지 카운트를 갱신하는 방식을 찾다가 
백그라운드 푸쉬를 이용해서 로컬 알림과 뱃지 카운트를 갱신이 가능하지 않을까 하여 위와 같은 방식으로 테스트하였습니다.

추가-앱이 종료된 상태에는 앱을 깨울 수 없습니다. 앱이 종료된 상태에서는 처리 할 수 없습니다.(참고 : https://developer.apple.com/documentation/uikit/uiapplicationdelegate/1623013-application :  However, the system does not automatically launch your app if the user has force-quit it. In that situation, the user must relaunch your app or restart the device before the system attempts to launch your app automatically again.)
