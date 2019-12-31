actix_lambda
============
[![Build Status](https://travis-ci.com/palfrey/actix_lambda.svg?branch=master)](https://travis-ci.com/palfrey/actix_lambda)
[![Crates.io](https://img.shields.io/crates/v/actix_lambda.svg)](https://crates.io/crates/actix_lambda)
[![MSRV: 1.39.0](https://flat.badgen.net/badge/MSRV/1.39.0/purple)](https://blog.rust-lang.org/2019/11/07/Rust-1.39.0.html)

Helper libraries for running/testing Actix servers under [AWS Lambda](https://aws.amazon.com/lambda/)

Currently, it just consists of a simple helper function `run` that will run the entire app as a lambda function, and `lambda_test` which will feed in a single [Application Load Balancer](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html) event into the Lambda app.

Usage
-----

```rust
fn app() -> App {
    return App::new()
        .route("/", Method::GET, root_handler);
        // More route handlers
}

fn main() {
    actix_lambda::run(app);
}

#[cfg(test)]
mod tests {
    #[test]
    fn lambda_test() {
        actix_lambda::test::lambda_test(main);
    }
}
```

In addition to the Rust code, there's also some Python work with [CloudFormation](https://aws.amazon.com/cloudformation/) and [Troposphere](https://github.com/cloudtools/troposphere/) to enable building stacks with this. To deploy this do the following:

1. Have a [CLI-configured AWS account](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html)
2. `rustup target add x86_64-unknown-linux-musl`
3. `brew install filosottile/musl-cross/musl-cross` (or do Linux-equivalent steps [to get a Musl cross-compiler](https://musl.cc/))
4. `mkdir .cargo && echo '[target.x86_64-unknown-linux-musl]\nlinker = "x86_64-linux-musl-gcc"' > .cargo/config`
3. `cargo build --release --target x86_64-unknown-linux-musl`
    * This may fail, especially if you're using something that uses OpenSSL. The notes at https://www.andrew-thorburn.com/cross-compiling-a-simple-rust-web-app/#compiling may well help you
3. cd &lt;copy of the helpers directory from here&gt;
4. `pip install -r requirements.txt`
5. `python cf.py <path to your app's root>`
    * This will make a CloudFormation stack named after your app, and then do some custom configuration of the TargetGroup and Listener for the ALB to [workaround an upstream bug](https://forums.aws.amazon.com/thread.jspa?threadID=294551)

You should now be able to run your app from the URL that the script spat out.

TODO
----
* Mechanisms for splitting up Actix apps into multiple Lambda functions
* Improved test functions with multiple varied requests
* Rewrite Troposphere work into pure CloudFormation with Rust once https://forums.aws.amazon.com/thread.jspa?threadID=294551 gets resolved