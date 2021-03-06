# PyO3-MultiProcessing

Rust bindings for Python multiprocessing module.

## Installation

``` toml
[dependencies]
pyo3-mp = "*"
```

## Usage

``` rust
use std::thread::sleep;
use std::time::Duration;

use pyo3::prelude::*;
use pyo3::wrap_pyfunction;

use pyo3_mp::Process;

/// A Python function implemented in Rust.
#[pyfunction]
fn foo(_py: Python, i: usize) -> PyResult<()> {
    println!("hello, number {}!", i);
    // This may be worked on each process!
    sleep(Duration::from_secs(1));
    println!("goodbye, number {}!", i);
    Ok(())
}

/// Converts the pyfunction into python object.
fn build_foo<'a>(py: Python<'a>) -> PyResult<Py<PyAny>> {
    Ok(wrap_pyfunction!(foo)(py)?.into_py(py))
}

fn main() -> PyResult<()> {
    Python::with_gil(|py| {
        // Let's get a sample python function.
        let foo = build_foo(py)?;

        let mut mp = Process::new(py)?;

        // Spawn 10 processes.
        for i in 0..10 {
            mp.spawn(&foo, (i,), None)?;
        }

        mp.join()
    })
}
```
