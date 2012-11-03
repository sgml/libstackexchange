#libstackexchange

Haskell interface to [Stack Exchange API v2.1][1]

##Getting started

You talk to StackExchange API server via `Request`s: quite simple data type
containing server host name, API call path, query parameters and so on. Simple
request would be like that:

```haskell
request = questions
```

Yeah, that simple. Next, `Request` is a `Monoid`. This means you
can easy construct and reuse arbitrarily complex requests:

```haskell
common = site "stackoverflow" <> key "12345678"
questions' = questions <> common
answers' = answers <> common
```

To get response from request, you need to send it to Stack Exchange:

```haskell
response = askSE $ questions'
```



## Authentication

Some API calls require authentication. The main goal is to get `access_token` via quite
[sophisticated process][3] ([serverside authentication skeleton][5] example might be helpful here. To get it to work you
must provide `client_id`, `redirect_uri` and `client_secret`). After that, you can freely call this priviledged API:

```haskell
response = askSE $ token "12345678" $ me <> key "12345678"
```

## Retrieving Data

StackExchange responses are, basically, wrapped [aeson][2] data structure, so you can
access data via ordinary aeson parsers:

```haskell
ghci> +m Control.Applicative Data.Aeson Data.Aeson.Types Data.Monoid Data.Text
ghci> data Title = Title Text deriving Show
ghci> instance FromJSON Title where parseJSON o = Title <$> (parseJSON o >>= (.: "title"))
ghci> qs <- askSE $ questions <> site "stackoverflow"
ghci> map (fromJSON . unSE) qs :: [Result Title]
[Success (Title "Playing encrypted video"),Success (Title "How do i get a If statment to read a multilined textbox?"), ...]
```

You can simplify it a bit by using `Functor Request` instance:

```haskell
ghci> askSE $ map (fromJSON . unSE) <$> questions <> site "stackoverflow" :: IO [Result Title]
[Success (Title "Playing encrypted video"),Success (Title "How do i get a If statment to read a multilined textbox?"), ...]
```

For _hardcore_ users familiar with [lens][4], there are `field` and
`fields` actions that cover the most common task of getting specific JSON field(s):

```haskell
qhci> (qs ^.. traverse) ^! traverse . field "title" :: [Text]
["Playing encrypted video", "How do i get a If statment to read a multilined textbox?", ...]
```

And if `field` and `fields` are not enough, full power of aeson is provided by `aeson` action.

 [1]: https://api.stackexchange.com/docs
 [2]: http://hackage.haskell.org/package/aeson
 [3]: https://api.stackexchange.com/docs/authentication
 [4]: http://hackage.haskell.org/package/lens
 [5]: https://github.com/supki/libstackexchange/blob/master/examples/server-side-authentication.hs