_This page contains implementation advice for view engines. It's work-in-progress_

* The _url_ of a Nancy request should always be _url-decoded_
* The query-string of a Nancy request should always be _url-encoded_
* Should set Content-Type of response sent by the host
* Should set Status of the response sent by the host
* Should set all cookies of the response sent by the host
* Should set all headers of the response sent by the host