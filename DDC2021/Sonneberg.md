# Sonneman123 writeup

Alright, so we're presented with the infomation that mean man sonne is back under the alias soppegaard.

A quick google search will lead you to his reddit profile, and this [post](https://www.reddit.com/user/soppegaard/comments/mql2af/livet_under_jorden/gxddbjs/?context=3)

First off, I (as many others), tried emailing the mail (soppegaard123@gmail.com), but after no response, and also no response on a reddit message I went further down the OSINT rabbit hole
I googled around for a tool to scour the internet for an email/alias, and fell over this [reddit post](https://www.reddit.com/r/OSINT/comments/gjct5q/using_an_email_address_to_find_social_media/)
After trying spokeo and other tools I found, I tried Spiderfoot, which lead me to this [tumblr profile](https://soppegaard123.tumblr.com/)

The [profile]((https://soppegaard123.tumblr.com/)) had two posts, and in one of them he spoke about his location with this great picture <br>
![Photo](https://64.media.tumblr.com/86f87a0143f29aef2cf9bef5b220e0c5/102e53b007b0c1db-a7/s2048x3072/8c89015e07468e75821767a764795f2333dfcdf6.jpg)
I tried sending the image to the guy on reddit, maybe the threat of knowing where he sorta is was enough to get the flag, but no.

Initially, I had written off the idea of using exiftool, as I assumed tumblr would strip picture metadata, but nope, after running
<b>`exiftool [image name]`</b>
I was presented with the flag
<b>DDC{Fl0pp3Rg44Rd_G3mm3R_s1G_p4_L0114nD}</b>
under image description