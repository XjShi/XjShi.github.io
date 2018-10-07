---
title: NSURLSession
date: 2015-03-30 21:28:17
tags: iOS
---
NSURLSession 这个类和与其有关联的其他类，提供一个通过 HTTP 下载的 API。这个 API 提供丰富的代理方法可以让你的app在挂起或者没有运行的时候，在后台下载。

用 NSURLSession API，你的应用可以创建一系列的 session，每一个 session 协调一组关联的数据转存任务。举个例子：你在协议一个网络浏览器，你的应用可以创建为每个
tab 或每个 window 创建一个 session。在每个 session 中，你的应用添加一系列的任务，每一个任务代表一个指定 URL 的请求。

三种类型 session：**default session** behave similary to other Foundation methods for downloading URLs,使用基于磁盘的 cache，并且在用户的钥匙串中存储 credentials，**ephemeral session** 不在磁盘上做缓存，所有的缓存、credential等都保持在 RAM 里，和 session 绑定；**background session** 它的行为跟 default session 相似，但它用一个单独的进程处理数据传输。把结果存在一个文件中，并且在app挂起、退出、崩溃的时候仍然继续转存数据。

在这些 session 中，你可以安排三种类型的任务：

　　**data task** 用 NSData 对象收发数据，Data task are intended for short, often interative request from you app to a server。可以一次返回一块数据，或者通过 completion handler 一次返回所有数据。Because data task do not store the data to file, they are not supported in background sessions。

　　**download task** retrieve data in the form of file, and support background downloads while the app is not running。

　　**upload task** send data(usuall in the form of a file),and support background uploads while the app is not running。

像大多数的网络 API 一样，NSURLSession 是高度异步的，它根据你调用的方法，用一种或两种方法返回数据：

* 一个 handler block，当转存完成或者出错的时候，给你的app返回数据。
* 通过调用你自定义的代理，当数据被收到的时候。
* 通过调用方法在自定义的代理上，当下载数据完成。

NSURLSession 的 API 除了传送信息给代理，也提供状态和过程属性。支持取消、重新开始、继续、挂起任务，并且有继续  挂起、取消或者失败的下载的能力。

## URL Sessioin Class Hierarchy

NSURLSession API包括下面这些类：

* NSURLSession:一个session对象
* NSURLSessionConfiguration: 初始化 session 时用得配置对象。
* NSURLSessionTask：session 中任务的基类。

	* NSURLSessionDataTask：任务用于获取一个 URL 的内容（作为 NSData）。
		* NSURLSessionUploadTask：上传文件，接着取回一个 URL 的内容（作为 NSData）。
	* NSURLSessionDownloadTask：获取一个 URL 的内容作为一个临时文件放在磁盘上。
	
除此之外，NSURLSession 的 API 提供了四个协议，你的app可以实现它们的代理方法以对 session 和 task 的行为进行更颗粒化（精细）的控制。

* NSURLSessionDelegate：处理 session-level 事件。
* NSURLSessionTaskDelegate：处理 task-level 事件，对所有任务类型可用。
* NSURLSessionDataDeleg：处理 task-level 事件，只对 data task 和 upload task 有用。
* NSURLSessionDownloadDelegate：处理 task-level 事件，只对 download task 有用。

最后，NSURLSession 的 API 用许多类，这些类也经常和其他 API 一起用（比如 NSURLConnection 和 NSURLDownload ）。

* NSURL：
* NSURLRequest：封装和URL request相关的元数据（metadata），包括URL，request方法，等等。
* NSURLResponse：封装和服务器对请求的响应有关的元数据，比如内容的MIME类型和长度。
	* NSHTTPURLResponse：为HTTP request添加额外的，比如响应头。
* NSCachedURLResponse:封装一个NSURLRespo对象（与服务器响应体的真实数据一起），目的是cache。

## Background Transfer Considerations

使用 background session 的时候，因为实际的传输由一个单独的进程来执行，也因为重启应用的代价相对比较大，因此，一些特性不可用，造成下面的限制：

* The session must provide a delegate for event delivery.(For uploads and downloads, the delegate behave the same as for in-process transfer.)
* 只支持HTTP/HTTPS协议，不支持自定义协议。
* 只支持upload task、download task，不支持data task。
* Redirects are always follows.
* If the background transfer is initiated while the app is in the background, the configuration object's discretionary property is treated as being true.

在 iOS 中，当后台传输完成或者需要资格(require credentials)，如果应用没有在运行，iOS 自动重启应用并调用`application:handleEventsForBackgroundURLSession:completionHandler:`方法在`UIApplicationDelegate`对象上。This call provides the identifier for the session that caused your app to be lunched. 你的app应该存储这个 completionHandler，用这个 identifier 创建 background configuration object，然后用这个配置对象创建session。The new session is automatically reassociated with on going background activity. Later, when the session finishes the last background download task, it sends the session delegate a URLSessionDidFinishEventsForBackgroundURLSession: message. Your session delegate should then call the stored completion handler.

In both iOS and OS X, when the user relaunches your app, your app should immediately create background configuration objects with the same identifiers as any session that had outstanding tasks when your app was last running, then create a session for each of those configuration objects. These new sessions are similary automatically reassociated with ongoing backgorund activity.

当你的app被挂起的时候，如果任何任务完成，这个任务的代理的`URLSession:downloadTask:didFinishDownloadingToURL:`  方法被调用。

Similarly, if any task requires credentials, the NSURLSession object calls the delegate’s `URLSession:task:didReceiveChallenge:completionHandler:` method or `URLSession:didReceiveChallenge:completionHandler:` method as appropriate. 

## 创建和配置

NSURLSession API提供的配置选项:

* Private storage support for caches,cookies,credentials,and protocols in a way that is specific to a single session.
* Authentication,tied to a specific request(task) or group of requests(session)。
* File uploads and downloads by URL,which encourage separation of the data (the file's contents)from the metadata(the URL and settings)
* Configuration of the maximum number of connections per host
* Per-source timeouts that are triggered if an entire resource cann't be downloaded in a certain mount of time
* Minimum and maximum TLS version support
* Custom proxy dictionaries
* Control over cookie policies
* Control over pipelining bahavior

----
博客搬家时的题外话：使用 NSURLSession 完全可以实现断点下载、后台下载，而且非常好用，需要仔细读读 URL Loading System 和 App 的生命周期相关文档，然后留意一个方法`getTasksWithCompletionHandler:`。别看某“大神”的“关于下载管理的探讨”讨论了。