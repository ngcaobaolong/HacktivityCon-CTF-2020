**HacktivityCon CTF - Template Sack: http://jh2i.com:50023**

Category : web, brute

----------------------------------------

We introduced with a web site, mostly static and a hint in the source **<!-- TODO: Finish developing the admin section and add functionality to manage the templates --\>**

No js loaded, no input box. So i looked around and found a jwt cookie. Throw it in jwt.io, we have 
**{
  "typ": "JWT",
  "alg": "HS256"
}** as header, and **{
  "username": "guest"
}** as body. I tried to set the alg to none but no luck, so i fired up johntheripper and start bruteforce the key. I thought it would took a while so i need look deeper, but the secret pop-up : **supersecret**
Throw it in jwt.io, we can change the cookie into whatever we want. So i modified the username to admin, encode it with the key and replace the current one.
And we got an admin page. There is a lot of link, but mostly returned with 404 page. But this page seemed weird. It reflect the url on to the page, and the challenge name is Template Shack so maybe template injection ? I tried that and it worked.
![templateinjection](https://scontent.fhan2-2.fna.fbcdn.net/v/t1.15752-9/116403034_295906118160053_5117863088692375563_n.png?_nc_cat=106&_nc_sid=b96e70&_nc_ohc=EQudZJJ8jf0AX9Z9-G7&_nc_ht=scontent.fhan2-2.fna&oh=f3e75f87df40a2fa4768753974839e28&oe=5F50F7AF)
So after some fuzzing using payload from https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Template%20Injection#jinja2, we knew that it currently using jinja2. So all we need to do left was find a payload that would result in RCE. I used this :
```
http://jh2i.com:50023/admin/%7B%%20for%20x%20in%20().__class__.__base__.__subclasses__()%20%%7D%7B%%20if%20%22warning%22%20in%20x.__name__%20%%7D%7B%7Bx()._module.__builtins__['__import__']('os').popen(request.args.input).read()%7D%7D%7B%endif%%7D%7B%endfor%%7D?input=cat%20flag.txt
```
And the flag got reflected:

![flag](https://scontent.fhan2-5.fna.fbcdn.net/v/t1.15752-0/p280x280/116264299_1407700809430461_432005348538169082_n.png?_nc_cat=109&_nc_sid=b96e70&_nc_ohc=Bsla3_11jHMAX8KusZy&_nc_ht=scontent.fhan2-5.fna&oh=d0bce4232e15e0d381ce5a75a5bdce64&oe=5F4DCD0C)

Simple challenge, but i dislike the bruteforcing part. It's kind of hit or miss.
Kudos to @vudq to bruteforce the key for me. My pc suck lol.
