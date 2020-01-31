---
layout: post
title: "Burp Suite with SQLMap on Kali Linux"
featured-img: bap_extension
category: [Guides]
---

This is a quick tutorial on how to use SQLMap within Burp Suite **(*Community* and *Pro Edition*)**. This is extremely useful when doing security evaluations, bug bounties, and penetration tests. There are quite a few tutorials across the Internet on how to do this, but I found most of them to be old and/or fragmented.

## Installing Jython Standalone

Jython is the JVM implementation of python. To utilize Burp extensions in Python, you must have the **standalone** version installed on your machine:

1. Install jython [standalone](http://search.maven.org/remotecontent?filepath=org/python/jython-standalone/2.7.1/jython-standalone-2.7.1.jar):

![jython](/assets/img/posts/jython.png)

2. Point Burp to the location of your previously downloaded standalone jython file under Extender->Options->Python Environment. As you can see, mine is located in the /usr/local/bin directory:

![load_burp_env](/assets/img/posts/load_burp_env.png)

## Installing SQLiPy extension in Burp

1. Download the SQLiPy extension in Burp under Extender->BApp Store. I already have the extensions installed, but if you don't you should see an 'Install' button where my 'Reinstall' button is located:

![bap_extension](/assets/img/posts/bap_extension.png)


## Using the SQLiPy extension in Burp

To use the burp SQLpi extension you must first start the SQLMap API server. To do so, head over to the SQLiPy->SQLMap API tab. From here you can specify the *Listen IP* (I would keep this at 127.0.0.1), the *Listen Port*, the *Python directory** (automatically populated), and the *SQLMap API* (automatically populated):

![sqlmap_api_off](/assets/img/posts/sqlmap_api_off.png)

Click the *Start API* button:

![sqlmap_api_on](/assets/img/posts/sqlmap_api_on.png)

Once the server has successfully started, you can start scanning requests that you see through your proxy tab. **Note**, this will work with HTTP GET **and** POST requests:

![sqlipy_send](/assets/img/posts/sqlipy_send.png)

After you send a request to the SQLiPy Scanner (SQLiPy->SQLMap Scanner) you can configure all your scan options, such as threads, DBMS Backend, Delay, etc. At a minimum, you will need to specify the parameter you want to test. Once you have configured any additional options required, click the scan button at the bottom of the window:

![scan_opts](/assets/img/posts/scan_opts.png)

You can see live updates of the results in the SQLiPy->SQLMap Logs tab:

![sql_logs](/assets/img/posts/sql_logs.png)


## Conclusion

I hope this serves as a useful reference for quickly setting up Burp Suite with SQLMap. Having them both in the same environment has been a lifesaver! 


## References:
* https://support.portswigger.net/customer/portal/articles/2791040-using-burp-with-sqlmap
* https://www.jython.org/download.html


