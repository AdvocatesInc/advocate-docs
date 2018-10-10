# Tracking Conversions

The Advocate platform includes tools to enable gathering detailed metrics on the performance of campaign assets from click to conversion. This document is intended for developers tasked with implementing metrics collection.

These collected metrics are viewable throughout the site. Including them is, however, disabled by default on new campaigns, so make sure to [contact us](mailto:admin@advocate.gg) when you're ready to start tracking them.

## Technical Requirements

Enabling conversion tracking requires some simple modifications to your website. When a viewer visits an Advocate campaign link, the originating link is included in the parameters of the target URL (the one you host). 

1. The implementing website must extract and store the value of the URL parameter `advref` when a visitor loads the page, then, *when the conversion occurs*, e.g. the customer clicks "Confirm Order", 
2. send a POST request to `http://api.adv.gg/v1/conversion/` with a JSON encoded body like `{'source': <value of advref>}.

### Example

The following script shows an example of a single page conversion tracking frontend implementation. In this case, the extraction of the referring source and the POSTing of the conversion take place in a direct sequence, i.e. both occur when the page loads and executes the script. In most cases, your implementation will consist of two parts, as noted above.

```
// env configuration
const CONVERSION_EVENT_ENDPOINT = 'http://api.adv.gg/v1/conversion/'

// get url params
const  getParameterByName = (name, url) => {
    if (!url) url = window.location.href;
    name = name.replace(/[\[\]]/g, '\\$&');
    const regex = new RegExp('[?&]' + name + '(=([^&#]*)|&|#|$)'),
        results = regex.exec(url);
    if (!results) return null;
    if (!results[2]) return '';
    return decodeURIComponent(results[2].replace(/\+/g, ' '));
}

// example helper function
const  setCookie = (cvalue, exdays, cname='advogg') => {
    const d = new Date();
    d.setTime(d.getTime() + (exdays*24*60*60*1000));
    const  expires = "expires="+ d.toUTCString();
    document.cookie = cname + "=" + cvalue + ";" + expires + ";path=/";
}

// example helper function
const getCookie = (cname='advogg') => {
    const name = cname + "=";
    const decodedCookie = decodeURIComponent(document.cookie);
    const ca = decodedCookie.split(';');
    for(let i = 0; i <ca.length; i++) {
        let c = ca[i];
        while (c.charAt(0) == ' ') {
            c = c.substring(1);
        }
        if (c.indexOf(name) == 0) {
            return c.substring(name.length, c.length);
        }
    }
    return "";
}

// step 1, above
const ref = getParameterByName('advref')
setCookie(ref, 1)

// step 2, above
const post_body = {
    source: getCookie()
}
// success handler for illustration only. The POST is, from conversion tracking perspective, a fire-and-forget request
const post_success = (response) => console.log(response.status, response.json)
fetch(CONVERSION_EVENT_ENDPOINT, {
    method: "POST",
    mode: "cors", // no-cors, cors, *same-origin
    headers: {
        "Content-Type": "application/json"
    },
    body: JSON.stringify(post_body), // body data type must match "Content-Type" header
})
    .then(post_success)
    .catch(error => console.error(`Fetch Error =\n`, error));
```
