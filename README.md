**Deploying a Go Application to Google Cloud**

For businesses, maintaining competitiveness and productivity is paramount. When embarking on application development, efficiency in scaling, cost reduction, rapid innovation, and ensuring reliability and security are top priorities. For Go developers, Google Cloud Platform (GCP) offers a comprehensive suite of tools and services tailored for deploying and managing applications efficiently.

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

In the "example/app" directory, create a new file named “main.go”, and paste the following code into it:

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

    // Start HTTP server.
    log.Print("listening on port 8080")
    if err := http.ListenAndServe(":8080", nil); err != nil {
        log.Fatal(err)
    }
}

func handler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprint(w, "Hello World!")
}
```

As you can see, the previous code creates a basic web server that listens on the port 8080.

Once you're done with the go code, add a new file named "Dockerfile", and paste the following content into in:

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

This is done to containerize the Golang application.

Then, let's use the Github Actions to automate repetitive tasks. In this case, I'll just add an action to check the code quality.

To get this part done, in the "example/app" directory, create a ".github/workflows" directory. The directory must have this exact name in order for Github to detect any Github Actions workflows that it contains.

In the ".github/workflows" directory, create a file with the ".yml" extension, and paste the following content into in:

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

Finally, enable dependency tracking, with the following command:

    go mod init example/app

Now, the sample app is ready to be pushed into the repository, and deployed.

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

Set a name and description.

Choose the option “Github - Continuously deploy from a repository.”

Set up with Cloud Build.

- Choose the repository you have created previously.
- Choose a branch.
- And choose “Dockerfile” as your “Build Type”, and set the source location.

That's all.

Now, you only need to wait a few minutes until the deployment is completed.

**Conclusion**

Once the service is created, a first deployment is done automatically. The URL of the application will be visible in the "Cloud Run Service Details" screen. Keep in mind that every change pushed to the "main" branch into the Github repository will trigger an action that runs linters to look for issues in the Go code. This is why you had implemented "Github Actions" that is a continuous integration and continuos delivery (CI/CD) platform. Also, every change pushed to the "main" branch will trigger a new deployment, so you probably want to add some restrictions here to manage a bit better when a new version of your application should be deployed.

This is just a small sample of all you could do with Go, and technologies like "Google Cloud", "Docker", and "Github".

Nothing more to add for the moment, thanks for reading!