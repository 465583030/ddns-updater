# ddns-updater
> "Who needs professional hosting for a website? Just get a Raspberry Pi, some web server software, and do it yourself."  
> "But I have a dynamic ip--I can't set my DNS records to my home broadband router without always having to check to see if they updated my ip."  
> "Well, did you try **ddns-updater**?"  
> "My head hurts. I don't want to have this conversation anymore."

**ddns-updater** will automatically update your Dynamic DNS records according to the configuration details you provide. It should be compatible with most DNS hosts who offer this service. Some examples include Google Domains and Namecheap, but there are many others. If they use a normal RESTful API, which takes updates via HTTP GET requests, then it should work fine provided your system has the required commandline tools installed ([see section below](#requirements)). I'm working on compatibilty with POST requests as well as the Cloudflare API.

## Requirements
**ddns-updater** relies on a few external commands on Unix-like systems. Among them, `curl`, `dig`, and `ping`. While it's almost certain that your system has `curl` and `ping` installed, it may not have `dig`. `dig` is a popular DNS probing tool and is installed with the **dnsutils** package. **ddns-updater** will automatically check to see if it's installed, but if it isn't, you can get it any number of ways. For Debian Linux-based systems, just run `apt-get install dnsutils`. For other Linux distros, a similar method most likely exists. Check your documentation. On Macs, `dig` is standard.

## Configuration
A sample configuration file has been created and can be used as a basis for your own. The structure is simple: Each key and value is separated by at least one space, key/value pairs are separated by line breaks, as are individual host blocks (the `hostname` and all other configuration fields). The only way the script knows one host and its associated details from another is by the `hostname` field, which is why it must always appear **first** in a host block. The order of the other fields, i.e. `update-url`, `secs-between`, and `failure-limit` are not important. Once again, the `hostname` field must always be first in the block. You may also skip lines for your own clarity, as well as use comments, which are just lines of text with a `#` at the beginning.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Field&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; | Description 
--- |--- 
hostname | The fully qualified domain name (FQDN) you wish to update: e.g. `subdomain.yourdomain.com`
update-url | The actual URL needed to perform the update. You can get this from your Dynamic DNS provider. The sample configuration file includes examples from Google Domains and Namecheap as well as links to documentation on those respective sites. If your URL requires your current IP address, just use `0.0.0.0` instead and the script will automatically replace it with your current IP.
~~success-pattern~~ (removed) | ~~This is an optional regular expression pattern to be matched against the response from your DDNS provider upon update request in order to determine whether the update was successful. If this option is left out, the system will simply check for a `200 OK` HTTP response code, then wait for as long as specified in the `secs-between` field before polling again. The script automatically checks the DNS records of your authoritative nameserver to determine if the update was successful. This option adds extra, but probably unnecessary, verification. Please note, the regex pattern must be POSIX Extended type. I would urge users to test their pattern with a successful response page before using it.~~ I'm removing this field because it just makes the script needlessly complicated. It is unimportant, if not unreliable, to make use of some arbitrary response page issued by your DDNS provider. The only thing that matters is that the record is up to date. **ddns-updater** will simply wait for as long as specified by the `secs-between` field before polling again. It will then check with your hostname's authoritative nameservers to see if they have updated. If not, it will issue another request. This is why it is necessary to set the `secs-between` field to, at minimum, 120, or 2 minutes. Otherwise, the script may issue redundant requests.
secs-between | Yes, I'm well aware of how it sounds. But actually, this variable determines how often the script will poll. If left out, the default value is 300 seconds, or 5 minutes. I would strongly suggest you don't set this to be any less than 60, since DDNS providers may have policies against that and if violated, could possibly prevent any further updates to that domain. Check with your provider to see how often you can poll. The default value should be good in most circumstances, although it could mean that your site will be unreachable for 5+ minutes.
failure-limit | Specifies how often an update request can fail before the script quits trying. If and when all hosts specified in the configuration reach their respective limits, the script will exit.


## Installation
You can just run **ddns-updater** from the commandline by typing `/path/to/ddns-updater -f /path/to/config-file`. This, however, causes **ddns-updater** to run the foreground, occupying your terminal window. Instead, you can send it to the background by adding a `&` to the end of the command, i.e. `/path/to/ddns-updater -f /path/to/config-file &`.

Ideally, however, you want **ddns-updater** to run automatically whenever your system starts up. There are a number of ways to do this depending on your specific OS. On Debian Linux-based systems, including Ubuntu, the easiest way to do it is just to add the above command directly to your `/etc/rc.local` file. Better yet, run the command as a specific user (perhaps for security reasons or whatever), such as:
```
su someuser -c "/path/to/ddns-updater -f /path/to/config-file &"
```
It may also be advisable to use the `nohup` command along with the `&` to prevent possible hangups caused by `someuser` logging in and out of the system. So:
```
su someuser -c "nohup /path/to/ddns-updater -f /path/to/config-file &"
```
On a Mac, you can just create a plain text file, calling it, say, `start-ddns-updater` and writing `/path/to/ddns-updater -f 'path/to/config-file &'` to this file. Then, go to your Terminal, and make `start-ddns-updater` executable by issuing this command: `chmod +x /path/to/start-ddns-updater`. Finally, go to `System Preferences > Users & Groups > Login Items` and add `start-ddns-updater` to your list of login items. Done.

On other OSes, similar methods exist. Check your documentation.
