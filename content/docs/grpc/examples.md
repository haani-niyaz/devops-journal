# GRPC Examples

## Setup

### Install dependencies

{{< highlight bash >}}
go get -u google.golang.org/grpc
go get -u github.com/golang/protobuf/protoc-gen-go
{{< / highlight >}}

### Environment setup

{{< highlight bash >}}
export GOPATH=/Users/haani/go
export GOBIN=/Users/haani/go/bin
export PATH=$PATH:$GOBIN
{{< / highlight >}}


## Run

{{< highlight bash >}}
protoc greet/greetpb/greet.proto --go_out=plugins=grpc:.
{{< / highlight >}}

## Handling Errors

The following packages must be imported:

- Server side:

{{< highlight go >}}
fn := req.GetFirstName()
if fn == "" {
        return nil, status.Errorf(codes.InvalidArgument,
            "Input cannot be empty")
    }
{{< / highlight >}}

- Client side:

{{< highlight go >}}
res, err := c.Greet(context.Background(), r)
    if err != nil {
        respErr, ok := status.FromError(err)
        if ok {
            // Expected error
            log.Println(respErr.Message)
        } else {
            // Framework error
            log.Panic("something went wrong %v", err)
        }
    }
    }
{{< / highlight >}}


### Worth Taking Another Look

- https://dev.to/techschoolguru/implement-unary-grpc-api-in-go-4cdj
- https://stackoverflow.com/questions/43167762/how-to-return-an-array-in-protobuf-service-rpc
- https://medium.com/pantomath/how-we-use-grpc-to-build-a-client-server-system-in-go-dd20045fa1c2
- https://stackoverflow.com/questions/46906014/how-to-implement-transformation-of-a-struct-to-a-similar-struct-in-go
