---
layout: post
title: "java处理https请求txt文件并写回"
date: 2024-11-13
---
为了提高程序的性能，我们可以使用多线程来并发地发送 HTTP 请求。Java 提供了多种方式来实现多线程，例如使用 `Thread` 类、实现 `Runnable` 接口，或者使用更高级的 `ExecutorService` 来管理线程池。

在这个场景下，使用 `ExecutorService` 线程池是一个很好的选择，因为它可以有效地管理线程池，并确保线程的复用和资源的合理利用。

### 多线程实现思路：
1. **使用线程池**：我们将每个 HTTP 请求的处理放在一个单独的任务中，这些任务由线程池中的线程并行执行。
2. **使用 `Callable` 代替 `Runnable`**：`Callable` 可以返回值，这使得我们可以获取每个请求的响应并处理它。
3. **确保线程安全**：由于多个线程可能会同时操作文件，我们需要确保写入文件时的线程安全。可以使用 `synchronized` 或 `ConcurrentLinkedQueue` 等方式来保证线程安全。

### 具体实现

```java
import java.io.*;
import java.net.HttpURLConnection;
import java.net.URL;
import java.util.concurrent.*;
import java.util.List;
import java.util.ArrayList;

public class MultiThreadedHttpRequest {
    private static final int THREAD_POOL_SIZE = 10; // 设置线程池大小

    public static void main(String[] args) {
        String inputFilePath = "requests.txt";
        String outputFilePath = "responses.txt";

        // 使用 ExecutorService 来管理线程池
        ExecutorService executorService = Executors.newFixedThreadPool(THREAD_POOL_SIZE);
        List<Future<String>> futures = new ArrayList<>(); // 存储每个任务的返回结果

        try (BufferedReader reader = new BufferedReader(new FileReader(inputFilePath))) {
            String url;
            // 读取文件中的 URL 并提交任务
            while ((url = reader.readLine()) != null) {
                if (!url.trim().isEmpty()) {
                    // 使用 Callable 任务来异步获取响应
                    Callable<String> task = () -> sendHttpRequest(url);
                    futures.add(executorService.submit(task)); // 提交任务
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        }

        // 处理任务结果并写入输出文件
        try (BufferedWriter writer = new BufferedWriter(new FileWriter(outputFilePath))) {
            for (int i = 0; i < futures.size(); i++) {
                Future<String> future = futures.get(i);
                String response = future.get(); // 获取每个请求的响应
                writer.write("URL " + (i + 1) + ": " + "\n");
                writer.write(response + "\n");
                writer.write("--------------------------------------------------\n");
            }
        } catch (IOException | InterruptedException | ExecutionException e) {
            e.printStackTrace();
        } finally {
            // 关闭线程池
            executorService.shutdown();
        }

        System.out.println("所有请求已完成，响应已写入 " + outputFilePath);
    }

    /**
     * 发送 HTTP 请求并返回响应内容
     */
    private static String sendHttpRequest(String urlString) {
        StringBuilder response = new StringBuilder();
        try {
            URL url = new URL(urlString);
            HttpURLConnection connection = (HttpURLConnection) url.openConnection();
            connection.setRequestMethod("GET");
            connection.setConnectTimeout(5000);
            connection.setReadTimeout(5000);

            int status = connection.getResponseCode();
            try (BufferedReader in = new BufferedReader(new InputStreamReader(
                    (status < HttpURLConnection.HTTP_BAD_REQUEST) ? connection.getInputStream() : connection.getErrorStream()))) {
                String inputLine;
                while ((inputLine = in.readLine()) != null) {
                    response.append(inputLine);
                }
            }
        } catch (IOException e) {
            response.append("Error: " + e.getMessage());
        }
        return response.toString();
    }
}
```

### 代码解释：

1. **线程池 (`ExecutorService`)**：
   - 使用 `Executors.newFixedThreadPool(THREAD_POOL_SIZE)` 创建一个固定大小的线程池，`THREAD_POOL_SIZE` 是线程池的大小，你可以根据实际需求调整线程池的大小。
   - 将每个 HTTP 请求的处理任务封装成 `Callable<String>`，因为 `Callable` 可以返回值，我们将响应结果作为 `String` 返回。

2. **提交任务 (`executorService.submit(task)`)**：
   - 对于每个 URL，创建一个 `Callable` 任务，并提交到线程池中。线程池将负责将任务分配给线程进行执行。

3. **获取结果**：
   - 使用 `Future<String>` 来获取每个任务的结果。`future.get()` 方法会阻塞直到任务完成并返回结果。

4. **写入文件**：
   - 在所有任务完成后，将每个 URL 的响应写入到 `responses.txt` 文件中。每个响应之间用分隔线隔开。

5. **异常处理**：
   - 捕获和处理可能的异常，包括网络请求的 `IOException` 和线程执行中的 `InterruptedException` 或 `ExecutionException`。

6. **线程池的关闭**：
   - 在所有任务完成后，通过 `executorService.shutdown()` 关闭线程池。

### 优化点：
1. **线程池的管理**：
   - 通过合理设置 `THREAD_POOL_SIZE`，可以平衡并发性能和系统负载，避免线程池过大会导致资源浪费，过小则可能无法充分利用 CPU。
   
2. **任务超时**：
   - 可以通过 `Future.get(long timeout, TimeUnit unit)` 设置任务的超时时间，防止某些请求由于网络问题或服务器问题导致永远阻塞。

3. **响应结果格式化**：
   - 在写入文件时，你可以进一步格式化响应内容，或者对 HTTP 状态码进行详细处理。

### 线程安全：
在多线程环境中，写入文件的操作可能存在竞争条件问题（多个线程同时写入文件）。为了保证线程安全，这里使用了 `BufferedWriter` 来串行地写入文件。如果需要处理更高并发的场景，可以使用 `synchronized` 或者线程安全的队列（如 `ConcurrentLinkedQueue`）来缓冲响应数据，然后按顺序写入文件。

### 总结：
通过使用 `ExecutorService` 和多线程，我们可以并行发送 HTTP 请求，显著提高程序的性能，特别是当需要处理大量请求时。此实现不仅保证了每个请求的结果被正确处理和写回文件，同时利用线程池的方式也能有效管理线程的创建和销毁，避免过多的资源消耗。