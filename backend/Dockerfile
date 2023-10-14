# multistage 
### build stage
FROM golang:alpine3.18 as builder

# set the working directory
WORKDIR /app

# copy go.mod/go.sum files
COPY go.mod go.sum ./
RUN go mod download

# copy the app source code
COPY ./ ./

# build
RUN go build -o main ./cmd/api


### run stage
FROM alpine:latest

# set the working directory
WORKDIR /app

# copy the built binary
COPY --from=builder /app/main .

EXPOSE 8081
CMD ["./main"]
