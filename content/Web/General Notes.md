
Sometimes an attack isn't working, not because you're doing the technique wrong, but because one of the tools you're using as part of your attack doesn't work the way you think. For example, doing a file upload attack with a PNG file, and always failing a MIME check even though you're matching the magic bytes check: that's because PNGs have other required elements throughout the image, so we need another format that only requires magic bytes. 

Another example: you're doing an XXE attack, but only get an html file rather than the php file you're asking for. That means you need to provide the full web path to get the php file before modifications. 


Who'd have thought that html forms require input tags in order to put get parameters?