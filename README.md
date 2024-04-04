**Deploying a Go Application to Google Cloud**

In this post, I'll show you how to set up a Go application using Cloud Build, ensuring that changes to your source repository are automatically built into container images in Artifact Registry and deployed to Cloud Run.

**Before proceeding**

Let's address a question you may have: What are the benefits of deploying applications to Google Cloud Run?

The following are some of the ones I would highlight:

- **Fully Managed Serverless Platform:** Cloud Run provides a fully managed serverless environment, freeing you from the infrastructure management. This allows you to focus on developing and deploying your applications.

- **Containerized Workloads:** Cloud Run supports containerized workloads, which gives you flexibility and portability in how you build and deploy your applications.

- **Automatic Scaling:** Cloud Run automatically scales your applications based on incoming requests, which can help you save on costs by only paying for resources when they're in use.

- **Pay-per-use Billing:** It means you only pay for the resources your application consumes, billed by the number of requests and the duration of each request.

**Keep in mind**

At the conclusion of this post, assuming you are going to diligently follow each step, you should have set up a Google Cloud Platform project, a service, and a GitHub repository to host your Go codebase.

So, to begin with, you need:
- **Go installed.**
- **Project on Google Cloud Platform.**
- **Billing account for your project.**

**Let's move on**

**Step 1: Writing a Golang app**

In your GoPath, create an “example/app” directory.

In the “app” directory, create a “main.go”.

```go
package main

import (
    "fmt"
    "log"
    "net/http"
    "os"
)

func main() {
    log.Print("starting server...")
    http.HandleFunc("/", handler)

    // Determine port for HTTP service.
    port := os.Getenv("PORT")
    if port == "" {
        port = "8080"
        log.Printf("defaulting to port %s", port)
    }

    // Start HTTP server.
    log.Printf("listening on port %s", port)
    if err := http.ListenAndServe(":"+port, nil); err != nil {
        log.Fatal(err)
    }
}

func handler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprint(w, "Hello World!")
}
```

In the “app” directory, create a Dockerfile.

```dockerfile
# Use the official Go image as the base image
FROM golang:latest

# Set the working directory inside the container
WORKDIR /app

# Copy the Go module files
COPY go.mod ./

# Download the dependencies
RUN go mod download

# Copy the rest of the application code
COPY . .

# Build the Go application
RUN go build -o main .

# Set the entry point command to run the built binary
CMD ["./main"]
```

Then, create a ".github/workflows" directory. The directory must have this exact name in order for Github to detect any Github Actions workflows that it contains.

In the ".github/workflows" directory, create a file with the ".yml" extension.

```yaml
name: Linting

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Go Linting
        run: |
          curl -sSfL https://raw.githubusercontent.com/golangci/golangci-lint/master/install.sh | sh -s v1.57.2
          bin/golangci-lint --version
          bin/golangci-lint run
```

And finally, enable dependency tracking, with the following command:

    go mod init example/app

**Step 2: Creating a repository**

Create a Github repository.

Now, from your local machine, create a new repository, with the following commands:

    cd example/app
    echo "# repository name" >> README.md
    git init
    git add .
    git commit -m "first commit"
    git branch -M main
    git remote add origin https://github.com/[account_name]/[repository_name].git
    git push -u origin main

**Step 3: Creating a service**

In Cloud Run, create a service.

Set a name, and description.

Choose the option “Github - Continuously deploy from a repository.”

Set up with Cloud Build.

- Choose the repository you have created previously.
- Choose a branch.
- And choose “Dockerfile” as your “Build Type”, and set the source location.

That's all.

Now, you only need to wait a few minutes until the deployment is completed.

**Conclusion**

...

Thanks for reading!