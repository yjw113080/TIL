

```
URL url = new URL(newUrls);
HttpURLConnection urlConnection = (HttpURLConnection) url.openConnection();

InputStream inputStream = new BufferedInputStream (urlConnection.getInputStream());
BufferedReader reader = new BufferedReader(new InputStreamReader(inputStream,"UTF-8"),8);

StringBuilder sb = new StringBuilder();
String line = "";
while((line = reader.readLine())!=null) {
    sb.append(line + "\n");
}
```


* String 타입에 그대로 +를 통해 문자열을 완성하게 되면, 그 단계 마다 새 문자열이 생성된다.
* StringBuffer와 StringBuilder 는 같은 인스턴스에 계속 append 하는 방식이다. <br />
수십번 String이 더해지는 경우에는 각 String의 주소값이 stack에 쌓이고 클래스들은 Garbage Collector가 호출되기 전까지 heap에 지속적으로 쌓이게 되기 때문에 메모리 관리에는 치명적이다.
* StringBuilder는 여러 쓰레드들이 동시에 접근할 수 있다. 
* StringBuffer는 multi thread환경에서 다른 값을 변경하지 못하도록 하므로 web이나 소켓환경과 같이 비동기로 동작하는 경우가 많을 때는 StringBuffer를 사용하는 것이 안전할 것이다. <br />
출처: https://novemberde.github.io/2017/04/15/String_0.html