1.多个线程并行执行,主线程阻塞等待所有子线程执行完(CountDownLatch实现)
源码:
```java
import java.util.concurrent.CountDownLatch;


public class CountDownLatchUtil {


    public static void allOf(Runnable... runnableArray) {
        if (arrayIsEmpty(runnableArray)) {
            return;
        }
        CountDownLatch countDownLatch = new CountDownLatch(runnableArray.length);
        for (Runnable runnable : runnableArray) {
            //  建议使用线程池
            new Thread(() -> {
                runnable.run();
                countDownLatch.countDown();
            }).start();
        }
        try {
            countDownLatch.await();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    private static <T> boolean arrayIsEmpty(T[] array) {
        return array == null || array.length == 0;
    }
}
``` 
 使用示例:
![image](https://github.com/5pyx55CG5ri4/5pyx55CG5ri4.github.io/assets/50692323/9c458e5e-6dbc-4e98-b47a-59714052dc6e)

后面发现Java8提供了CompletableFuture也可以实现,但是感觉用起来没有自己写的简单
2.多线程并行操作集合
源码:
```java
import java.util.ArrayList;
import java.util.Collection;
import java.util.List;
import java.util.function.Consumer;


public class AsyncHandleUtil {

    private static final int DEFAULT_PAGE_SIZE = 1000;

    public static <T> void asyncHandle(List<T> list, Consumer<T> action, int pageSize) {
        if (collectionIsEmpty(list)) {
            return;
        }
        int totalPage = totalPage(list.size(), pageSize);
        for (int i = 0; i < totalPage; i++) {
            int pageNum = i + 1;
            //  建议使用线程池
            new Thread(() -> {
                List<T> newDataList = getListPaging(list, pageNum, pageSize);
                for (T t : newDataList) {
                    action.accept(t);
                }
            }).start();
        }
    }


    public static <T> void asyncHandle(List<T> list, Consumer<T> action) {
        asyncHandle(list, action, DEFAULT_PAGE_SIZE);
    }


    private static int totalPage(int totalCount, int pageSize) {
        if (pageSize == 0) {
            return 0;
        } else {
            return totalCount % pageSize == 0 ? totalCount / pageSize : totalCount / pageSize + 1;
        }
    }


    private static boolean collectionIsEmpty(Collection<?> collection) {
        return collection == null || collection.isEmpty();
    }


    private static <T> List<T> getListPaging(List<T> list, int pageNum, int pageSize) {
        if (collectionIsEmpty(list)) {
            return new ArrayList<>(0);
        }
        int startIndex = (pageNum - 1) * pageSize;
        int endIndex = pageNum * pageSize;
        int total = list.size();
        int pageCount;
        int num = total % pageSize;
        if (num == 0) {
            pageCount = total / pageSize;
        } else {
            pageCount = total / pageSize + 1;
        }
        if (pageNum == pageCount) {
            endIndex = total;
        }
        return list.subList(startIndex, endIndex);
    }
}
```
使用示例:
![image](https://github.com/5pyx55CG5ri4/5pyx55CG5ri4.github.io/assets/50692323/728ade16-98fe-4c5a-bcce-de7e6bed5cd9)

跟Java8的并行流操作差不多,但是更灵活