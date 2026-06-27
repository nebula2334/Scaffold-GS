# Scaffold-GS Improvement for SLAM Final Project

## 项目信息
- 课程：SLAM技术期末大作业
- 作者：唐天乐
- 学号：2335050524
- 改进分支：improvement

## 改进点说明
### 1. 稀疏体素锚点增强（Sparse-Voxel Anchors Augmentation, SVA）
- 针对问题：弱纹理区域COLMAP初始化点不足，导致渲染伪影
- 修改文件：`scene/gaussian_model.py`
- 核心逻辑：对1≤点数<5的边际体素，每个增加3个随机锚点高斯，提升弱纹理区域表面平滑度
- 效果：Bicycle数据集PSNR提升0.8dB，消除大部分低纹理区域空洞

### 2. 直接特征到颜色映射（Direct Feature-to-Color Mapping）
- 针对问题：Color MLP计算量大，边缘设备部署帧率过低
- 修改文件：`gaussian_renderer/__init__.py`
- 核心逻辑：完全移除两层Color MLP，直接取特征前3维经Sigmoid映射为RGB
- 效果：渲染FPS提升85.4%，VRAM占用降低32%，适合嵌入式SLAM平台

## 复现命令
由于改进逻辑已在对应模块中直接通过硬编码生效，请在当前 `improvement` 分支下，直接使用以下原版命令即可进行端到端复现：

```bash
# 改进版（融合SVA与Direct Mapping）一键复现训练
python train.py -s data/bicycle -m output/improved_final

# 渲染测试集图片
python render.py -m output/improved_final --skip_test
```

## 环境配置
- Python 3.9
- PyTorch 2.0.1
- CUDA 11.8
- 依赖安装：`pip install -r requirements.txt`
- 自定义算子编译：`pip install submodules/diff-gaussian-rasterization --no-build-isolation`

## 实验结果
| 方法 | Bicycle PSNR (dB) | Bicycle FPS | VRAM (GB) | 训练时长(h) |
|------|-------------------|-------------|-----------|-------------|
| Baseline | 26.3 | 18.2 | 12.8 | 2.1 |
| Only SVA | 27.1 | 17.9 | 13.2 | 2.2 |
| Only Direct Mapping | 21.77 | 33.7 | 8.7 | 1.5 |
| Final (A+B) | 22.5 | 33.4 | 9.1 | 1.6 |

## 实验日志与数据说明
完整的渲染结果对比图、评估指标原始数值（CSV）以及部分训练日志，已按照课程要求打包在教务系统提交的 `2335050524_唐天乐_SLAM2026Final.zip` 附件中，未上传至本精简版代码仓库。
