# video.requestVideoLoadCallback() Explainer

# Introduction
Today,  most of web sites are using `<video src="{file address}">` for media playback, but there is no easy way to access to response information such as video data, HTTP code, error details and so on.

We propose a new `HTMLVideoElement.requestVideoResponseCallback()` method to allow web developers to get response information after <video> sent request to server.


# Use cases

If error occurs in server side, we can get error information and HTTP error code from <video> (e.g., HTTP Code 500 for server internal error). 

Sites can use this API to analyse the stability and quality of media resource providers such as Amazon, Akamai, Aliyun.

Web developers can use this API to calculate download speed of current resource, if speed too slow to play current resolution, we can notice user to switch to lower quality for a fluent playback.




# Proposed API

```Javascript
callback VideorRequestCallback = void(Response request);
callback VideoResponseCallback = void(Response response);

partial interface HTMLVideoElement {
    unsigned long requestVideoRequestCallback(VideoRequestCallback callback);
    void cancelVideoRequestCallback(unsigned long handle);
    unsigned long requestVideoResponseCallback(VideoResponseCallback callback);
    void cancelVideoResponseCallback(unsigned long handle);
};
```


# Example

```Javascript
  let video = document.createElement('video');

  let responseCallback = (response) => {
    const dataSize = response.headers.get('content-length');
    const responseCode = response.status;
    console.log(
      `get response code ${responseCode}` +
      `with ${dataSize}byte of video data payloaded`);

    video.requestVideoResponseCallback(responseCallback);
  };

  video.requestVideoResponseCallback(responseCallback);
  video.src = 'foo.mp4';
```



# Implementation Details

* `video.requestVideoResponseCallback()` callbacks one-off, and must be called again to get the next response.
* Since request <video> send would some times have client side error occurs, for example network error. In that case, `responseCallback` won't be triggered
* `responseCallback` is not designed to play with MediaSource and BlobURL, if we are using these API to feed media data to `<video>` `responseCallback` won't be triggered
