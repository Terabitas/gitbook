# REST API in less than 3 minutes

Before you begin, please follow [Setting up environment](setting_up_environment.md) section if you haven't done that yet.

This document will guide you through the steps required to create a working REST API with `nildev`. If you want just get this up and running - keep reading. If you want same guide but with more details - please read [REST API service explained](rest_api_service_explained.md).

## 1. Create REST API project skeleton

Create `servicex.json` file at your home directory:
```
{
  "ApiName":"servicex",
  "ApiNameAllCapital":"SERVICEX",
  "ProjectPath":"github.com/your_org/servicex",
  "ApiNameExported":"Servicex",
  "TableName":"sessions",
  "Org":"your_org",
  "FullAuthor":"Your Name <your@email.com>",
}
```

Create new project:

```
nildev create --config=~/servicex.json github.com/your_org/servicex
```

## 2. Build REST API docker image

Run the following:

```
nildev build github.com/your_org/servicex
```

## 3. Running your service

Create `local.env` file in `$GOPATH/src/github.com/your_org/servicex`:

```
ND_DATABASE_NAME=Sessions
ND_MONGODB_URL=mongodb://mongodb.authx.nildev:27017/nildev
ND_ENV=dev
```

Now run your service:
```
docker-compose -f docker-compose.yml up -d
```

## 4. Test it!

```
curl -X POST http://YOUR_DOCKER_MACHINE_IP/api/v1/custom-register/github?userName=nildev -v
```

# What's next

Please read [REST API service explained](rest_api_service_explained.md) guide which will explain individual steps.
