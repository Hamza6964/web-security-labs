Bypassing XSS Filters: Exploiting PortSwigger’s “Reflected XSS with Some SVG Markup Allowed” Lab
While working through PortSwigger's XSS labs, I came across a scenario where most tags and attributes were heavily filtered — but with a bit of digging, I discovered a clean path to trigger JavaScript.

### Initial Recon
The vulnerability existed in the search functionality, a classic entry point for reflected XSS. I began with the usual suspects:

<script>alert(1)</script> 
<img src=x onerror=alert(1)>

But the app responded with: "tag not allowed."
__check(image1)__


### Fuzzing for Allowed Tags (Burp Suite Intruder)
To identify which tags were still allowed, I used Burp Suite Intruder to brute-force a list of HTML/SVG tags inside the search input. I configured Intruder to cycle through each tag and monitored the HTTP responses.
After analyzing the responses, I found that only two tags consistently returned a 200 OK:
<svg> And <animateTransform>
This hinted that SVG-based injection was possible.
__(check image 2)__

### Testing for Allowed Attributes
I followed the same approach to test event handler attributes (onload, onerror, onclick, etc.). Most were blocked — except:
**onbegin**
This was the key.

### The Final Payload
Using only the allowed elements and attributes, I crafted a minimal payload that successfully triggered an alert:

<svg><animateTransform onbegin=alert(1) attributeName=transform type=rotate from=0 to=1 dur=1s/></svg>

__(check image4)__

Why it works:

onbegin is a valid SVG event that fires when the animation starts.

animateTransform is a legitimate SVG element.

The alert is triggered as soon as the animation begins.

### Key Takeaways
Don't assume non-script tags are safe — SVG tags can be just as dangerous.

Event handlers like onbegin can be overlooked by filters but still execute JavaScript.

Burp Suite Intruder is powerful for fuzzing both tags and attributes in filtered environments.

This lab was a great example of how fine-grained filtering can still leave exploitable gaps if one overlooked tag or attribute is left open.

Let me know if you’ve pulled off any clever SVG-based XSS bypasses — would love to compare techniques.
