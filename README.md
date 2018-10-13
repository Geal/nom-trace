# nom-trace

This crate provides a way to trace a parser execution,
storing positions in the input data, positions in the parser
tree and parser results.

As an example, if you run the following code:

```rust
#[macro_use] extern crate nom;
#[macro_use] extern crate nom-trace;

//adds a thread local storage object to store the trace
declare_trace!();

pub fn main() {
  named!(parser<Vec<&[u8]>>,
    //wrap a parser with tr!() to add a trace point
    tr!(preceded!(
      tr!(tag!("data: ")),
      tr!(delimited!(
        tag!("("),
        separated_list!(
          tr!(tag!(",")),
          tr!(digit)
        ),
        tr!(tag!(")"))
      ))
    ))
  );

  println!("parsed: {:?}", parser(&b"data: (1,2,3)"[..]));

  // prints the last parser trace
  print_trace!();

  // the list of trace events can be cleared
  reset_trace!();
}
```

You would get the following result
```
parsed: Ok(("", ["1", "2", "3"]))
preceded        "data: (1,2,3)"

        tag     "data: (1,2,3)"

        -> Ok("data: ")
        delimited       "(1,2,3)"

                digit   "1,2,3)"

                -> Ok("1")
                tag     ",2,3)"

                -> Ok(",")
                digit   "2,3)"

                -> Ok("2")
                tag     ",3)"

                -> Ok(",")
                digit   "3)"

                -> Ok("3")
                tag     ")"

                -> Error(Code(")", Tag))
                tag     ")"

                -> Ok(")")
        -> Ok(["1", "2", "3"])
-> Ok(["1", "2", "3"])
```

Parser level is indicated through indentation. For each trace point, we have:

- indent level, then parser or combinator name, then input position
- traces for sub parsers
- `->` followed by the parser's result

This tracer works with parsers based on `&[u8]` and `&str` input types.
For `&[u8]`, input positions will be displayed as a hexdump.