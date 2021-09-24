# 06-substarte-homework

## 1.为template模块的do_something添加benchmark用例

### 1.1修改cargo.toml
添加
```bash
runtime-benchmarks = ["frame-benchmarking"]
```

在[dependencies]中添加
```bash
frame-benchmarking = { default-features = false, version = '3.0.0', optional = true }
```

在[feature]中添加
```bash
'frame-benchmarking/std',
```

### 1.2创建benchmarking.rs
```bash
//! Benchmark-demo pallet benchmarking.

#![cfg(feature = "runtime-benchmarks")]

use super::*;

use frame_benchmarking::{benchmarks, account};
use frame_system::RawOrigin;
use sp_std::prelude::*;

benchmarks!{
	do_something {
		let b in 1 .. 1000;
		let caller = account("caller", 0, 0);
	}: _ (RawOrigin::Signed(caller), b.into())
	verify {
		let value = Something::<T>::get();
		assert_eq!(value, b.into());
	}
}

#[cfg(test)]
mod tests {
	use super::*;
	use crate::mock::{new_test_ext, Test};
	use frame_support::assert_ok;

	#[test]
	fn test_benchmarks() {
		new_test_ext().execute_with(|| {
			assert_ok!(test_benchmark_do_something::<Test>());
		});
	}
}

```


### 1.3创建weights.rs
```bash

use frame_support::{traits::Get, weights::{Weight, constants::RocksDbWeight}};
use sp_std::marker::PhantomData;

/// Weight functions needed for pallet_benchmark_demo.
pub trait WeightInfo {
	fn do_something(b: u32, ) -> Weight;
}

/// Weights for pallet_benchmark_demo using the Substrate node and recommended hardware.
pub struct SubstrateWeight<T>(PhantomData<T>);
impl<T: frame_system::Config> WeightInfo for SubstrateWeight<T> {
	fn do_something(b: u32, ) -> Weight {
		(16_726_000 as Weight)
			// Standard Error: 0
			.saturating_add((8_000 as Weight).saturating_mul(b as Weight))
			.saturating_add(T::DbWeight::get().writes(1 as Weight))
	}
}

// For backwards compatibility and tests
impl WeightInfo for () {
	fn do_something(b: u32, ) -> Weight {
		(16_726_000 as Weight)
			// Standard Error: 0
			.saturating_add((8_000 as Weight).saturating_mul(b as Weight))
			.saturating_add(RocksDbWeight::get().writes(1 as Weight))
	}
}

```

## 1.4修改runtime目录
在runtime/src.lib.rs的frame_benchmarking::Benchmark<Block>的dispatch_benchmark函数中添加
```bash
add_benchmark!(params, batches, pallet_template, TemplateModule);
```

配置config
```bash
impl pallet_template::Config for Runtime {
	type Event = Event;
	type WeightInfo = pallet_template::weights::SubstrateWeight<Runtime>;
}
```
## 1.4编译和运行
```bash
cargo build --features runtime-benchmarks --release
```
运行node-template
```bash
./target/release/node-template benchmark  \
--chain dev --execution=wasm \
--wasm-execution=compiled \
--pallet pallet_template \
--extrinsic do_something \
--steps 20 \
--repeat 50
```

结果如下：

https://github.com/LeonP3ng/play-substrate/blob/master/2.%E8%BF%90%E8%A1%8C%E7%BB%93%E6%9E%9C.png

![avatar](https://github.com/LeonP3ng/play-substrate/blob/master/2.%E8%BF%90%E8%A1%8C%E7%BB%93%E6%9E%9C.png)

## 1.5将运行结果转为权重
```bash
/// # <weight>
/// - Base Weight: 26.75 us
/// - DB Weight: 1 write
/// # </weight>
#[pallet::weight(T::DbWeight::get().writes(1) + 27_000_000)]
pub fn do_something(origin: OriginFor<T>, something: u32) -> DispatchResultWithPostInfo {...}
```

# 2.选择node-template 生成chain spec

## 2.1生成chain spec

```bash
./target/release/node-template build-spec --disable-default-bootnode --chain local > customSpec.json
```

文件链接
https://github.com/LeonP3ng/play-substrate/blob/master/customSpec.json

## 2.2编码chain spec

```bash
 ./target/release/node-template build-spec --disable-default-bootnode --chain=customSpec.json --raw  > customSpec-raw.json
```

文件链接
https://github.com/LeonP3ng/play-substrate/blob/master/customSpec-raw.json