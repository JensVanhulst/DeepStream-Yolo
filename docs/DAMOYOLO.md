# DAMO-YOLO usage

* [Convert model](#convert-model)
* [Compile the lib](#compile-the-lib)
* [Edit the config_infer_primary_damoyolo file](#edit-the-config_infer_primary_damoyolo-file)
* [Edit the deepstream_app_config file](#edit-the-deepstream_app_config-file)
* [Testing the model](#testing-the-model)

##

### Convert model

#### 1. Download the DAMO-YOLO repo and install the requirements

```
git clone https://github.com/tinyvision/DAMO-YOLO.git
cd DAMO-YOLO
pip3 install -r requirements.txt
pip3 install onnx onnxsim onnxruntime
```

**NOTE**: It is recommended to use Python virtualenv.

#### 2. Copy conversor

Copy the `export_damoyolo.py` file from `DeepStream-Yolo/utils` directory to the `DAMO-YOLO` folder.

#### 3. Download the model

Download the `pth` file from [DAMO-YOLO](https://github.com/tinyvision/DAMO-YOLO) releases (example for DAMO-YOLO-S*)

```
wget https://idstcv.oss-cn-zhangjiakou.aliyuncs.com/DAMO-YOLO/release_model/clean_model_0317/damoyolo_tinynasL25_S_477.pth
```

**NOTE**: You can use your custom model.

#### 4. Convert model

Generate the ONNX model file (example for DAMO-YOLO-S*)

```
python3 export_damoyolo.py -w damoyolo_tinynasL25_S_477.pth -c configs/damoyolo_tinynasL25_S.py --simplify --dynamic
```

**NOTE**: To change the inference size (defaut: 640)

```
-s SIZE
--size SIZE
-s HEIGHT WIDTH
--size HEIGHT WIDTH
```

Example for 1280

```
-s 1280
```

or

```
-s 1280 1280
```

#### 5. Copy generated files

Copy the generated ONNX model file and labels.txt file (if generated) to the `DeepStream-Yolo` folder.

##

### Compile the lib

Open the `DeepStream-Yolo` folder and compile the lib

* DeepStream 6.2 on x86 platform

  ```
  CUDA_VER=11.8 make -C nvdsinfer_custom_impl_Yolo
  ```

* DeepStream 6.1.1 on x86 platform

  ```
  CUDA_VER=11.7 make -C nvdsinfer_custom_impl_Yolo
  ```

* DeepStream 6.1 on x86 platform

  ```
  CUDA_VER=11.6 make -C nvdsinfer_custom_impl_Yolo
  ```

* DeepStream 6.0.1 / 6.0 on x86 platform

  ```
  CUDA_VER=11.4 make -C nvdsinfer_custom_impl_Yolo
  ```

* DeepStream 6.2 / 6.1.1 / 6.1 on Jetson platform

  ```
  CUDA_VER=11.4 make -C nvdsinfer_custom_impl_Yolo
  ```

* DeepStream 6.0.1 / 6.0 on Jetson platform

  ```
  CUDA_VER=10.2 make -C nvdsinfer_custom_impl_Yolo
  ```

##

### Edit the config_infer_primary_damoyolo file

Edit the `config_infer_primary_damoyolo.txt` file according to your model (example for DAMO-YOLO-S* with 80 classes)

```
[property]
...
onnx-file=damoyolo_tinynasL25_S.onnx
model-engine-file=damoyolo_tinynasL25_S.onnx_b1_gpu0_fp32.engine
...
num-detected-classes=80
...
parse-bbox-func-name=NvDsInferParseYoloE
...
```

**NOTE**: The **DAMO-YOLO** do not resize the input with padding. To get better accuracy, use

```
maintain-aspect-ratio=0
```

##

### Edit the deepstream_app_config file

```
...
[primary-gie]
...
config-file=config_infer_primary_damoyolo.txt
```

##

### Testing the model

```
deepstream-app -c deepstream_app_config.txt
```

**NOTE**: The TensorRT engine file may take a very long time to generate (sometimes more than 10 minutes).

**NOTE**: For more information about custom models configuration (`batch-size`, `network-mode`, etc), please check the [`docs/customModels.md`](customModels.md) file.