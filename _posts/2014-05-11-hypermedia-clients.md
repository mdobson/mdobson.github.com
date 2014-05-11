---
layout: post
title: FizzBuzz, Siren, and Hypermedia
---

Recently I've been doing quite a bit of work with [Siren Hypermedia](https://github.com/kevinswiber/siren) APIs. I've been working through implementing a client in Objective-C for interacting with that particular media type, and a good gut check was to use FizzBuzz as a Service by Stephen Mizell. Here are a couple screenshots of the app.

![Screen shot 1]({{site.url}}/assets/screenshot1.png)
![Screen shot 2]({{site.url}}/assets/screenshot2.png)

It's a very simple and straight forward app. It uses [SHMKit](https://github.com/mdobson/SHMKit) my implementation of a Siren client library.
This service was a great gut check for implementing something other than tests due to it's determinism, and ability to walk links or trigger actions
on the fizzbuzz state machine.

Here is the first bit of code that walks the Siren object graph. It simply waits for no more `next` links and prints out the results when done.
This method took about 101 requests to actually complete (counting the initial request for the root doc).

```
-(void)fizzbuzz:(SHMEntity *) entity {
    if ([entity hasLinkRel:@"next"]) {
        [self patchLink:entity andLinkRel:@"next"];
        [entity stepToLinkRel:@"next" withCompletion:^(NSError *error, SHMEntity *entity) {
            if (error) {
                NSLog(@"Next step err:%@", error);
            } else {
                NSLog(@"Value:%@", entity.properties[@"value"]);
                [self fizzbuzz:entity];
            }
        }];
    } else {
        //Completed
        self.indicator.hidden = YES;
        self.number.text = [NSString stringWithFormat:@"%@",entity.properties[@"number"]];
        self.value.text = entity.properties[@"value"];
    }
}
```

The second method is to use the `get-fizzbuzz-value` action. This allows us to trim down our requests to only 2! One for the original Siren document and one for triggering the action.

```
-(void)searchFizzbuzz:(id)sender {
    self.queryIndicator.hidden = NO;
    [self.queryIndicator startAnimating];
    NSString * number = self.fizzbuzzQuery.text;
    NSNumber * fizzbuzzValue = [NSNumber numberWithInt:[number integerValue]];
    NSDictionary *params = @{@"number": fizzbuzzValue};
    [self.fizzbuzz performActionWithFields:params andCompletion:^(NSError *error, SHMEntity *entity) {
        if (error) {
            NSLog(@"Error: %@!", error);
        } else {
            self.queryIndicator.hidden = YES;
            self.queryNumber.text = [NSString stringWithFormat:@"%@",entity.properties[@"number"]];
            self.queryValue.text = entity.properties[@"value"];
        }
    }];
}
```

Overall, this was a fun Saturday morning code project to try out. It was a great gut check for my hypermedia client, and a good way to see hypermedia in action.
