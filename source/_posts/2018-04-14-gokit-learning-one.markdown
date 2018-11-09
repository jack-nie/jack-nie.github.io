---
layout: post
title:  "go-kit学习笔记(一)"
date:   "2018-04-14"
keywords: ["golang", "go-kit"]
description: "go-kit学习笔记"
category: "Go"
tags: ["Go"]
---

本文是学习go-kit的开篇，主要参考资料是官方文档。

在前面的一篇文章中，已经介绍了基于go开发api服务需要哪些步骤，那么我们将来实践这些步骤。

首先创建一个interface，

    import "context"

    type StringService interface {
            Uppercase(context.Context, string) (string, error)
            Count(context.Context, string) int
    }

然后编写代码实现这个interface。

    import (
            "context"
            "errors"
            "strings"
    )

    type stringService struct{}

    func (stringService) Uppercase(_ context.Context, s string) (string, error) {
            if s == "" {
                    return "", ErrEmpty
            }
            return strings.ToUpper(s), nil
    }

    func (stringService) Count(_ context.Context, s string) int {
            return len(s)
    }

    var ErrEmpty = errors.New("Empty string”)

在go-kit中首选的消息机制是RPC，所以我们接下来定义request和response的结构，保存输入和输出的数据。

    type uppercaseRequest struct {
            S string `json:"s"`
    }

    type uppercaseResponse struct {
            V   string `json:"v"`
            Err string `json:"err,omitempty"` // errors don't JSON-marshal, so we use a string
    }

    type countRequest struct {
            S string `json:"s"`
    }

    type countResponse struct {
            V int `json:"v"`
    }

接下来就到了endpoint相关的逻辑了，在go-kit中，一个endpoint对应刚刚定义的interface中的方法，这里将要做的就是实现适配器将stringService中对应的方法转换成endpint.Endpoint类型。

    import (
            "context"
            "github.com/go-kit/kit/endpoint"
    )

    func makeUppercaseEndpoint(svc StringService) endpoint.Endpoint {
            return func(ctx context.Context, request interface{}) (interface{}, error) {
                    req := request.(uppercaseRequest)
                    v, err := svc.Uppercase(ctx, req.S)
                    if err != nil {
                            return uppercaseResponse{v, err.Error()}, nil
                    }
                    return uppercaseResponse{v, ""}, nil
            }
    }

    func makeCountEndpoint(svc StringService) endpoint.Endpoint {
            return func(ctx context.Context, request interface{}) (interface{}, error) {
                    req := request.(countRequest)
                    v := svc.Count(ctx, req.S)
                    return countResponse{v}, nil
            }
    }

接下来就到了transport层，这里我们需要将服务暴露出去，以便提供给外部调用，底层协议可以根据需要选择，这里采用http和json。

    import (
            "context"
            "encoding/json"
            "log"
            "net/http"

            httptransport "github.com/go-kit/kit/transport/http"
    )

    func main() {
            svc := stringService{}

            uppercaseHandler := httptransport.NewServer(
                    makeUppercaseEndpoint(svc),
                    decodeUppercaseRequest,
                    encodeResponse,
            )

            countHandler := httptransport.NewServer(
                    makeCountEndpoint(svc),
                    decodeCountRequest,
                    encodeResponse,
            )

            http.Handle("/uppercase", uppercaseHandler)
            http.Handle("/count", countHandler)
            log.Fatal(http.ListenAndServe(":8080", nil))
    }

    func decodeUppercaseRequest(_ context.Context, r *http.Request) (interface{}, error) {
            var request uppercaseRequest
            if err := json.NewDecoder(r.Body).Decode(&request); err != nil {
                    return nil, err
            }
            return request, nil
    }

    func decodeCountRequest(_ context.Context, r *http.Request) (interface{}, error) {
            var request countRequest
            if err := json.NewDecoder(r.Body).Decode(&request); err != nil {
                    return nil, err
            }
            return request, nil
    }

    func encodeResponse(_ context.Context, w http.ResponseWriter, response interface{}) error {
            return json.NewEncoder(w).Encode(response)
    }

至此，一个简单的基于go-kit的micro service已经全部开发完成了。
