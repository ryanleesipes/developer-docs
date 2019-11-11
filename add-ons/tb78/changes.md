# Changes in Thunderbird 69-70

This document tries to cover all the changes that may by needed to make add-ons compatible with Thunderbird 70. If you find stuff that is no longer working but is not yet on this list, ask for help and advice in the mozilla.dev.apps.thunderbird newsgroup or check our [communication channels](https://wiki.mozilla.org/Thunderbird/CommunicationChannels#If_you.27re_a_developer).

The changes are grouped by category and are listed in the order we became aware of them.

## Removed XUL elements

### &lt;textbox type="number"&gt;

Removed in TB70. Use

```markup
<html:input type="number">
```

### &lt;textbox&gt;

Removed completely in TB71. Use

```markup
<html:input>
```

## Changed API

### document.createElement\(\)

With TB69, the default namespace for createElement has switched from XUL to XHTML. So you can no longer create XUL elements with `document.createElement()` \(but XHTML elements\). To create XUL elements, use:

```javascript
document.createXULElement();
```

### nsISocketTransportService

Bug [1558726](https://bugzilla.mozilla.org/show_bug.cgi?id=1558726) introduced a breaking change to the `nsISocketTransportService` interface:

The first parameter used to be an array and the second one its length. This length parameter has been dropped, causing all subsequent arguments to shift by one. Furthermore, to create a default socket, you now have to pass an empty array as first parameter, instead of null.

To stay backward compatible, check the argument count of "createTransport". In case it is 4 is is the new interface, in case it is 5 you got the old interface. Alternatively, you may also check if the version of Thunderbird is 69 or later.

```text
   if (transportService.createTransport.length === 4)
      return transportService.createTransport(((secure) ? ["starttls"] : []), host, port, proxyInfo);

    if (transportService.createTransport.length === 5) {
      if (secure)
        return transportService.createTransport(["starttls"], 1, host, port, proxyInfo);

      return transportService.createTransport(null, 0, host, port, proxyInfo);
   }

   throw new Error("Unknown Create Transport signature");
```

### nsICookie2 

Merged into nsICookie.

### window.QueryInterface\(\)

With TB70, window objects don't need to be and cannot be QI'ed to nsIDOMChromeWindow, nsIDOMWindow and nsIInterfaceRequestor. You can just remove its usage, this is backwards compatible to TB 68. See the fix applied to [enigmail](https://gitlab.com/enigmail/enigmail/commit/cebbbc2a2f3d380e34ae8e4600f66e7986a0889d#6a57d3cb2d256247fb48986b448b6c289de8ffea).

{% hint style="info" %}
This does not apply to `nsIXULWindow`, which for example is used in `nsIWindowMediatorListener.onOpenWindow(xulWindow)`.
{% endhint %}

Equally, TreeColumns and TreeContentView don't need to be and cannot be QI'ed to nsITreeView.

### Services.scriptSecurityManager.createCodebasePrincipal\(\)

Renamed. Use

```javascript
Services.scriptSecurityManager.createContentPrincipal()
```

### nsIStringBundle.formatStringFromName\(\)

The 3rd parameter has been droped, which was the length of the array passed in as the 2nd parameter. See the [patch applied to Thunderbird itself](https://bug1557829.bmoattachments.org/attachment.cgi?id=9071511).

## Changes to the Preference System

### Removal of onsyncfrompreference and onsynctopreference Attributes

With TB70, these two attributes have been removed from XUL elements like `textbox`, `radio`, `checkbox` and friends. This is no longer supported:

```markup
<textbox
    id="my-textbox"
    preference="some.preference.name"
    onsynctopreference="return onToPref();"
    onsyncfrompreference="return onFromPref();"/>
```

Most of the XUL based preferences system has been long removed and had to be replaced by either using the [`preferencesBindings.js`](https://developer.thunderbird.net/add-ons/updates/tb68#less-than-prefwindow-greater-than-less-than-prefpane-greater-than-less-than-preferences-greater-than-and-less-than-preference-greater-than) [wrapper](https://developer.thunderbird.net/add-ons/updates/tb68#less-than-prefwindow-greater-than-less-than-prefpane-greater-than-less-than-preferences-greater-than-and-less-than-preference-greater-than) or by manually reimplementing that functionality using the`nsIPrefService`. 

The second approach is not affected by this change, as it has full control over the load and save process for all its preferences. The[`preferencesBindings.js`](https://developer.thunderbird.net/add-ons/updates/tb68#less-than-prefwindow-greater-than-less-than-prefpane-greater-than-less-than-preferences-greater-than-and-less-than-preference-greater-than) [wrapper](https://developer.thunderbird.net/add-ons/updates/tb68#less-than-prefwindow-greater-than-less-than-prefpane-greater-than-less-than-preferences-greater-than-and-less-than-preference-greater-than) has been updated and now supports:

```javascript
Preferences.addSyncFromPrefListener(
     document.getElementById("my-textbox"),
     function() { return onFromPref() });

Preferences.addSyncToPrefListener(
     document.getElementById("my-textbox"),
     function() { return onToPref() });     
```

