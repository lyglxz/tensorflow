<!-- Autogenerated by mlir-tblgen; don't manually edit -->
### `-tf-executor-to-functional-conversion`: Lifts tf_executor.island inner ops from a tf_executor.graph
This pass converts tf_executor.graphs consisting of only tf_executor.islands and
a tf_executor.fetch into a sea of nodes consisting of TensorFlow Dialect ops by
lifting such ops out of a tf_executor.graph's tf_executor.islands. If V1 control
flow ops are present in a tf_executor.graph, an error will be returned.

For example, the following:

```mlir
func @my_fn(%arg0: tensor<i32>, %arg1: tensor<i32>) -> (tensor<i32>, tensor<i32>) {
  %graph_results:2 = tf_executor.graph {
    %island_0_result, %island_0_control = tf_executor.island {
      %identity = "tf.Identity"(%arg0) : (tensor<i32>) -> tensor<i32>
      tf_executor.yield %identity : tensor<i32>
    }
    %island_1_result, %island_1_control = tf_executor.island {
      %identity_n:2 = "tf.IdentityN"(%arg1, %island_0_result) : (tensor<i32>, tensor<i32>) -> (tensor<i32>, tensor<i32>)
      tf_executor.yield %identity_n#0
    }
    tf_executor.fetch %island_0_result, %island_1_result : tensor<i32>, tensor<i32>
  }
  return %graph_results#0, %graph_results#1 : tensor<i32>, tensor<i32>
}
```

will be transformed into:

```mlir
func @my_fn(%arg0: tensor<i32>, %arg1: tensor<i32>) -> (tensor<i32>, tensor<i32>) {
  %identity = "tf.Identity"(%arg0) : (tensor<i32>) -> tensor<i32>
  %identity_n:2 = "tf.IdentityN"(%arg1, %identity) : (tensor<i32>, tensor<i32>) -> (tensor<i32>, tensor<i32>)
  return %identity, %identity_n#0 : tensor<i32>, tensor<i32>
}
```
### `-tf-shape-inference`: Simple Shape Inference on TensorFlow Dialect

#### Options
```
-max-iterations : Maximum shape inference iterations
```
