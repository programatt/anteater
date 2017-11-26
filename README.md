# Anteater - CI Check Framework

![anteater](http://i.imgur.com/BPvV3Gz.png)

Description
-----------

Anteater is an open framework using standard regular expressions to ensure
unwanted strings, filenames and binaries are not included in any patch or Pull
request to any of your git repositories.

You tell anteater exactly what you don't want to get merged, and anteater looks
after the rest.

With a few simple steps it can be easily implemented into a CI / CD workflow
with tooling such as Travis CI, CircleCI, Gitlab CI/CD and jenkins, gerrit and
possibly others.

It is currently used in the Linux Foundations project ['OPNFV'](https://opnfv.org) as means to
provid automated security checks at gate, but as shown in the examples below,
it can be used for much more such as ensuring depreciated objects etc are not
included in a patch / pull request.

Why would I want to use this?
-----------------------------

Anteater has many uses.

First, as mentioned, it can be set up to block strings and files with a
potential security impact or risk. This could include private keys, a shell
history, aws credentials etc.

It is especially useful at ensuring that elements used in a staging /
development enviroment are not merged into a production enviroment.

Let's take a look at an example:

```
apprun:
  regex: app\.run\s*\(.*debug.*=.*True.*\)
  desc: "Running flask in debug mode can give away sensitive data"
```

The above will match code where a flask server is set to running in debug mode
`` app.run(host='0.0.0.0' port=80 debug=true)``, which can be typical to a
developers enviroment and mistakenly staged into production.

For a rails app, this could be:
``\<%=.*debug.*%>``

Even more simple, look for the following in most logging frameworks:

``log\.debug``

How about credential file that would cause a job loss if ever leaked into
production? Anteater works with file names too.

Perhaps:

``jenkins\.plugins\.publish_over_ssh\.BapSshPublisherPlugin\.xml``

Or even..

```
- \.pypirc
- \.gem\/credentials
- aws_access_key_id
- aws_secret_access_key
- jenkins\.plugins\.publish_over_ssh\.BapSshPublisherPlugin\.xml
```

If your own app has its own secrets / config file, then its very easy to
add your own regular expressions. Everything is set using YAML formatting,
so no need to change anteaters code. It's hardly machine learning, blockchain
based AI tech, no, but RegEx is something we all know and can put to use.

Depreciated functions, classes etc
----------------------------------

Another use is for when our project depreciates an old function, yet developers
still make pull requests using the old function naming:

```
depreciated_function:``
  regex: depreciated_function\(.*\)
  desc: This function was depreciated in release X, use Y function.
```

What if I get false postives?
-----------------------------

Easy, you set a RegExp to stop the match , kind of like RegExp'ception.

Example:

Let's say we want to stop use of MD5:

```
md245:
  regex: md[245]
  desc: "Insecure hashing algorithm"
```

This then incorrectly gets matched to the following:

``mystring = int(amd500) * 4``

We set a specific ignore RegEx, so it matches and then is unmatched by the
ignore entry.

``mystring.=.int\(amd500\).*``

Block Binaries
--------------

Let's say for example, you have some image files, compiled objects or any form
of binary. Well we would not want one of those to get malicously replaced
with an infected blob would we? Before you scoff, it does happen. There have
been occurances of where developers SSH keys have been stolen and hackers have
self approved patches and managed to get trojan files on a production server.

With anteater, if you pass the argument ``--bincheck``, every binary causes a
CI build failure on the related Pull Request. It is not until a sha256 checksum
is set within anteater's YAML files, that the build is allowed to pass.

For example:

```
$ anteater --bincheck --project myproj --patchset /tmp/patch
Non Whitelisted Binary file: /folder/to/repo/images/pal.png
Please submit patch with this hash: 3aeae9c71e82942e2f34341e9185b14b7cca9142d53f8724bb8e9531a73de8b2
```
Let's enter the hash::
```
binaries:
  images/pal.png:
    - 3aeae9c71e82942e2f34341e9185b14b7cca9142d53f8724bb8e9531a73de8b2
```
Run the job again::
```
$ anteater --bincheck --project myproj --patchset /tmp/patch
Found matching file hash for: /folder/to/repo/images/pal.png
```
For more details and indepth documentation, please visit [readthedocs](http://anteater.readthedocs.io/en/latest/)

Last of all, if you do use anteater, I would love to know (twitter: @lukeahinds)
and pull requests / issues are welcome!
