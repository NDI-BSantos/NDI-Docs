# Stream Validation

To help highlight some potential problems, the NDI HX SDK will validate many aspects of streams that are passed to it. It is designed so that it will display the associated error on STDOUT if your stream is incorrect and then terminate the stream (to highlight that it is not a valid stream).

\[Obviously, this can make it quite important to monitor STDOUT in your implementation]{.underline}.

Important Note: To ensure that NDI HX works correctly across all devices and that a great end-user experience can be expected, it is very important that you take significant time testing to confirm that your implementation fully complies with this document and works well in practice.
