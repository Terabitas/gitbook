# REST API and SPA in less than 10 minutes

Please follow [Setting up environment](setting_up_environment.md) section if you haven't done that yet.

In this document we will create a working solution which will consist of REST API and SPA. In next chapters we will dive in more details how all this works, but purpose of this document is to get you up and running.

## Create REST API project skeleton

Create `authx-service.json` file at your home directory and replace `your_org` with your own.
```
{
  "ApiName":"authx",
  "ApiNameAllCapital":"AUTHX",
  "ProjectPath":"github.com/your_org/authx",
  "ApiNameExported":"Authx",
  "TableName":"sessions",
  "Org":"your_org",
  "FullAuthor":"Your Name <your@email.com>",
}
```

Create new project (replace `your_org` with your own):

```
nildev create --config=/path/to/authx-service.json github.com/your_org/authx 
```

## Create SPA project skeleton

Create `my-spa.json` file at your home directory and replace `your_org` with your own.
```
{
  "Name":"MySPA",
  "ProjectPath":"github.com/your_org/my-spa",
  "Org":"your_org",
  "FullAuthor":"Your Name <your@email.com>",
}
```

Create new project (replace `your_org` with your own):

```
nildev create --config=/path/to/my-spa.json github.com/your_org/my-spa 
```

## Build REST API docker image

Run the following (replace `your_org` with your own):

```
nildev build github.com/your_org/authx
```

## Build SPA docker image

Run the following (replace `your_org` with your own):

```
nildev build github.com/your_org/my-spa
```

## Running your service



### Test it!

Here [http://127.0.0.1:8080/my-spa](http://127.0.0.1:8080/my-spa) your `SPA` should be available. 
