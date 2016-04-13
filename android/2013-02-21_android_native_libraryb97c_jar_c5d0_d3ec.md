# Android Native Library를 JAR에 포함시켜 Dynamic Loading하기

## 배경
Android Library Project는 이는 라이브러리에 리소스 및 외부 라이브러리가 포함되어 있을 수 있기 때문에 일반적인 Java Library 처럼 별도의 JAR로 Export할 수 없다.


## 일반적인 경우에 App에 Native Library 사용법
먼저, Native Library를 앱에 포함시킬 경우, 다음과 같은 형태로 앱에 포함시키게 된다.

	/libs/armeabi/xxxx.so

위의 .so 파일은 앱이 설치되고 난 후에 다음의 경로로 옮겨지게 된다.

	/data/data/<app-id>/lib/xxxx.so
	
이때, Native Library를 기반으로 만들어진 JAR 라이브러리라면 so 파일과 함께 JAR도 추가시켜줘야할 것이다.

	/libs/armeabi/libxxxx.so
	/libs/xxxx.jar

xxxx.jar 내부에서는 xxxx.so를 사용하기 위해 아래와 같이 라이브러리를 불러올 것이다.

	class NativeLibraryTest {
	    // ...
	    static {
	        System.loadLibrary("xxxx");
	    }
	    // ...
	}

만약 xxxx.jar가 xxxx.so를 포함하고 있고, 내부적으로 .so를 불러오게 할 수만 있다면 최종 App 개발자에게 제공하는 라이브러리 파일은 xxxx.jar로 심플해질 것이다.

## Native Library를 JAR에 포함시키는 방법
앞에서 살펴본 것과 같이, 일반적으로 Java 코드에서 Library를 불러오기 위해서는, 아래 두 API를 사용하게 된다.

	System.load(libraryPath);
	System.loadLibrary(libraryName);

라이브러리에 단순히 libs 또는 lib에 .so 라이브러리를 넣고 JAR로 Export한 후, 이를 앱에서 사용하게 되면 JAR에 Native Library가 포함되어 있다는 이유로 앱을 만들수 없다고 메시지가 나오게 된다.
아래는 lib 라는 폴더 안에 .so 파일을 넣어 만든 JAR를 앱에서 사용했을 경우 발견되는 Message다.

	lib/ is reserved for NDK libraries. The following libraries were found:
	lib/armeabi/libxxxx.so

이를 회피하고자, 간단히 .so 파일 확장자를 바꾸고, directory 이름을 임의로 바꿔보았다.

	/src/net.daum.adam.test/NativeLibraryTest.java <- libxxxx를 불러오는 코드
	/native-lib/armeabi/libxxxx

이렇게 Library를 JAR로 묶은 후, `/native-lib/armeabi/libxxxx`를 임시파일로 만들어서 이를 `System.load(libraryPath);`로 사용하려고 했으나
임시파일까지는 만들어졌지만 실제 라이브러리로 불러올 수가 없었다.

하지만, 임의로 라이브러리를 앱의 캐시 영역으로 복사를 하고 사용해보니 정상적으로 불러올 수 있었다.
이때, App의 캐시 영역의 Path를 알기 위해서는 Context 객체의 `Context.getCacheDir()` API를 사용했다.


	class NativeLibraryTest {
	    public NativeLibraryTest(Context context) {
    	    String libraryPath = context.getCacheDir().getAbsolutePath() + "/libxxxx.so";

	        try {
	            //
	            // JAR에 /native-lib/libxxxx 파일을 앱 캐시 영역으로 복사한다.
	            //
	            NativeUtil.installLibrary("/native-lib/libxxxx", libraryPath);
	        } catch (IOException e) {
		        e.printStackTrace();
        	}

	        System.load(libraryPath);
	   }
	}

위에서 사용한 NativeUtil 클래스는 아래와 같이 작성하였다.

	public class NativeUtil {
	  private static final String TAG = "NativeUtil";
	
	  public static void installLibrary(String src, String dst) throws IOException {
	    Log.d("NativeUtil", "Install Native Library from " + src + " to " + dst);
	
	    byte[] buffer = new byte[1024];
	
	    InputStream is = NativeUtil.class.getResourceAsStream(src);
	    
	    if (is == null) {
	      throw new FileNotFoundException("File " + src + " was not found inside JAR");
	    }
	
	    OutputStream os = new FileOutputStream(dst);
	    
	    int readBytes;
	    
	    try {
	      int readBytes;
	      while ((readBytes = is.read(buffer)) != -1) {
	        int readBytes;
	        os.write(buffer, 0, readBytes);
	      }
	    } finally {
	      os.close();
	      is.close();
	    }
	  }
	}


이렇게 하고 나니 정상적으로 기존 라이브러리에서 Native Library를 불러올 수 있게 되었다.

## 이슈는 없을까?
위와 같이 작성했을 경우, 예상되는 이슈는 다음과 같다.

* NativeLibraryTest 객체를 생성할 경우에 매번 Library를 캐시영역으로 복사한다.
* 앱 캐시 영역을 제거하면 라이브러리가 제거된다.
	* 하지만, 객체 생성시에 라이브러리를 캐시영역으로 복사한 후에 라이브러리를 불러오기 때문에 메모리에서 내려갔다고 해도 다시 올라오게 될 것이다.
	
## 참고 사이트
* [Packaging Android Libraries with Native Code](http://blog.crittercism.com/2012/08/22/packaging-android-libraries-with-native-code/)
* [How to bundle a native library and a JNI library inside a JAR?](http://stackoverflow.com/questions/2937406/how-to-bundle-a-native-library-and-a-jni-library-inside-a-jar)