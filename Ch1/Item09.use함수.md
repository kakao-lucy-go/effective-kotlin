close메소드를 사용해서 명시적으로 닫아야 하는 리소스들이 있다.

- InputStream, OutputStream
- java.sql.Connection
- java.io.Reader(FileReader, BufferedReader, CssParser)
- java.new.Socker, java.util.Scanner

AutoCloeable을 상속받는 Cloeable 인터페이스를 구현한 리소스들인데, 레퍼런스가 없어지면 GC가 처리하는데 오래 걸리고 비용이 많이 든다. 따라서 사용이 끝나면 close 메소드를 호출하는데, 전통적으로는 try-finally 블록을 사용해서 처리했다. try 블록과 finally 블록에서 오류가 발생하면 그 블록에서 던진 오류만 throw 하는데 둘 다 던지도록 하기에는 코드가 길어져서 Closeable 객체에서 사용할 수 있는 use 함수를 사용할 수 있다.

```kotlin
val reader = BufferedReader(FileReader(path))
reader.use {
	return reader.lineSequence().sumBy { it.length }
}

File(path).useLines { lines ->
	return lines.sumBy { it.length } //단, 파일의 줄을 한 번만 사용할 수 있다.
}
```

```java
//use 함수 decompile
private static final Object use(Closeable $this$use, Function1 block) {
      int $i$f$use = 0;
      boolean var3 = false;
      Throwable exception = (Throwable)null;
      boolean var9 = false;

      Object var4;
      try {
         var9 = true;
         var4 = block.invoke($this$use);
         var9 = false;
      } catch (Throwable var11) {
         exception = var11;
         throw var11;
      } finally {
         if (var9) {
            InlineMarker.finallyStart(1);
            if (PlatformImplementationsKt.apiVersionIsAtLeast(1, 1, 0)) {
               closeFinally($this$use, exception);
            } else if ($this$use != null) {
               if (exception == null) {
                  $this$use.close();
               } else {
                  try {
                     $this$use.close();
                  } catch (Throwable var10) {
                  }
               }
            }

            InlineMarker.finallyEnd(1);
         }
      }

      InlineMarker.finallyStart(1);
      if (PlatformImplementationsKt.apiVersionIsAtLeast(1, 1, 0)) {
         closeFinally($this$use, exception);
      } else if ($this$use != null) {
         $this$use.close();
      }

      InlineMarker.finallyEnd(1);
      return var4;
   }
```
