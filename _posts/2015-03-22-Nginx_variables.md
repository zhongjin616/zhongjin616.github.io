hi there :-)

i am back again. today i will talk about Nginx variables

args VS variables
------------------

`ngx.arg[n]` often colabor with `set_by_lua*` which is read-only and holds the input arguments to the `config directives`:

`
    location /foo {
        set $a 32;
        set $b 56;
        set_by_lua $sum
            “return tonumber(ngx.arg[1]) + tonumber(ngx.arg[2])”
            $a $b;
        echo $sum
    }
`
they also can be refered by `ngx.var.a` and `ngx.var.sum`

query string which are specified in the url by a `?` often called arguments,
and nginx built-in variable  `args` contains the query string of url. when the url is : `http://localhost/test?a=0&b=1&c=2`
then the `args` will be “a=0&b=1&c=1”. besides arguments  can alos be referenced by `ngx.var.arg_XXX` respectively.
thus `ngx.var.arg_a` will be “0”.

And it can also contain the var in `location` which is matched by `regexp`. such as:

` location ~* ^/v1/(?<col>[^/]*)([/]?)(?<id>.*)$ `

thus we can use ngx.var.col and ngx.var.id to refer the string filtered by nginx,
which maybe something like `/v1/tokens/device_id01230` or `/v1/users/alice@molmc.com`


