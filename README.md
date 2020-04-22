# cbnet
implement cbnet with mmdetection

## Result

On COCO val2017

* `cascade_rcnn + dual res2net101 + dcnv2 + fpn + 1x`: 48.7 mAP
* `cascade_rcnn + dual resnet_vd200 + dcnv2 + nonlocal + fpn + 2.5x + soft-nms`: 52.2 mAP (weights are transfered from [CBResNet200-vd-FPN-Nonlocal](https://github.com/PaddlePaddle/PaddleDetection/blob/release/0.2/docs/MODEL_ZOO_cn.md) in paddle detection; 50.8 without soft-nms)

## backbone config example
```python
backbone=dict(
    type='CBNet',
    num_repeat=2,
    pretrained='/workspace/nas-data/checkpoint/imagenet/dual_res2net101_v1b_26w_4s-0812c246.pth',
    use_act=False,
    connect_norm_eval=True,
    backbone_type='Res2Net',
    depth=101,
    scale=4,
    baseWidth=26,
    num_stages=4,
    out_indices=(0, 1, 2, 3),
    norm_cfg=dict(type='SyncBN', requires_grad=True),
    dcn=dict(type='DCNv2', deformable_groups=1, fallback_on_stride=False),
    stage_with_dcn=(False, True, True, True),
    frozen_stages=1,
    style='pytorch')
```
To get the pretrained model on imagenet like `dual_xxx.pth`:
```
def make_pretrained_model(input_path, output_path, repeat_num=2):
    cp = torch.load(input_path)
    keys = list(cp['state_dict'].keys())
    for key in keys:
        for i in range(repeat_num):
            cp['state_dict']['cb{}.{}'.format(i + 1, key)] = cp['state_dict'][key]
        cp['state_dict'].pop(key)
    torch.save(cp, output_path)
```
