# 📏 Calculate Skeleton Graph Radius Error

Python script for estimating **skeleton / spatial graph radius error** by comparing graph-based radius values against effective radii measured from **cross-sectional-area-minimizing 2D reslices** through a binary vessel segmentation (`.tif`). Designed for evaluating how well spatial graph thickness values reflect vessel size in 3D vascular image data.

This repository contains one script:

* `calculate_radius_error_from_cross-sectional-area_minimizing_reslices.py`

---
<br><br>
## ⚙️ Environment Setup

You can install dependencies with pip:

```bash
pip install -U numpy scipy tifffile matplotlib opencv-python
```

---
<br><br>
## 🗂️ Script Overview

### 📊 `calculate_radius_error_from_cross-sectional-area_minimizing_reslices.py`
Measures effective vessel radii at sampled skeleton / spatial graph points by searching for the **2D reslice orientation that minimizes cross-sectional area** within a local subvolume, then compares that effective radius to the radius estimate stored in the spatial graph thickness field.

* **Input:** binary segmentation `.tif`, folder of Avizo SpatialGraph `.am` files
* **Output:** per-sample radius error table printed to terminal
* **Main comparison:** graph radius estimate (`t_cl`) vs measured effective radius (`r_eff`)

The script:

* reads a binary vessel segmentation
* computes a chamfer distance map (CDM)
* samples skeleton / spatial graph points
* extracts a local binary subvolume around each sampled point
* searches across reslice orientations to find the minimum-area cross-section
* computes an effective radius from the perimeter of that cross-section
* prints signed and absolute relative error for each sampled point

---
<br><br>
## 📐 Radius Error Overview

For each sampled graph point, the script compares:

* **Radius Estimate (`t_cl`)** — the radius value stored in the spatial graph thickness field
* **Measured Radius (`r_eff`)** — the effective radius derived from the cross-sectional-area-minimizing 2D reslice
* **CDM Value (`cdm_val`)** — the maximum local chamfer distance map value used to size the search region

The printed table includes:

```text
Sample  Radius Estimate (t_cl)  Measured Radius (r_eff)  CDM Value (cdm_val)  Signed Relative Error  Absolute Relative Error
```

Where:

* **Signed Relative Error** = `(t_cl - r_eff) / r_eff × 100`
* **Absolute Relative Error** = `|Signed Relative Error|`

---
<br><br>
## 🔍 Method Summary

### 1. Binary segmentation and CDM
The script reads a binary vessel segmentation and computes a **Chamfer Distance Map (CDM)**.

### 2. Point sampling
A set of skeleton / spatial graph points is sampled from the `.am` file.

### 3. Local subvolume extraction
For each point, a local binary subvolume is extracted. Its size is based on the local and nearby CDM values.

### 4. Minimum-area reslice search
A coarse-to-fine angular search is performed to find the 2D reslice orientation that minimizes cross-sectional area:

* coarse search: `45°`
* fine search: `5°`
* finest search: `1°`

### 5. Effective radius calculation
The central connected component of the best reslice is isolated, and its perimeter is measured. Effective radius is then calculated as:

```text
r_eff = perimeter / (2π)
```

### 6. Radius error reporting
The measured effective radius is compared to the graph thickness-based radius estimate for each sampled point.

---
<br><br>
## 🧭 Workflow

The expected workflow is:

1. **Start with a binary vessel segmentation** (`.tif`)
2. **Prepare one or more Avizo SpatialGraph files** (`.am`) whose radius values you want to evaluate
3. **Run** `calculate_radius_error_from_cross-sectional-area_minimizing_reslices.py`
4. **Inspect the printed radius error table** for each spatial graph file

---

### 🧪 Example Workflow

1. **Start with a binary vessel segmentation** (`.tif`)  
   This should be the same segmentation from which the spatial graph was derived, or the segmentation you want to use as the geometric reference.

2. **Provide one or more SpatialGraph files** (`.am`)  
   These should contain:
   * node coordinates
   * edge point coordinates
   * thickness values

3. **Set user inputs in the script**
   * `image_read_path`
   * `am_folder_read_path`
   * `voxel_size`
   * `sample_n`
   * `subvolume_edge_size_cdm_factor`

4. **Run the script**  
   It will:
   * load the segmentation
   * compute the CDM
   * loop through all `.am` files in the input folder
   * sample graph points
   * measure effective radii from minimum-area reslices
   * print a radius error table for each file

---

💡 **In short:**  
> **Segmentation** (`.tif`) → **SpatialGraph(s)** (`.am`) → **Sample graph points** → **Find cross-sectional-area-minimizing reslices** → **Measure effective radius** → **Compare against graph thickness radius**

---
<br><br>
## ⚠️ Important Interpretation Note

The accuracy of the measured re-slice radius depends on the **medialness** of the skeleton / spatial graph points.

If a sampled point lies close to the vessel centreline, the minimum-area reslice is more likely to approximate a true vessel cross-section. But if a point is offset toward the vessel edge, the search may find a lower-area slice by passing more **longitudinally** through the vessel rather than orthogonally across it.

This means:

* points with poor medial placement can produce artificially small measured cross-sections
* the resulting `r_eff` may underestimate the true local vessel radius
* apparent radius “error” may therefore reflect both:
  * radius inaccuracy, and
  * point placement away from the vessel medial axis

So this script is best interpreted as evaluating **radius estimate quality conditional on point placement**, rather than as a pure radius-only error metric.

---
<br><br>
## 🧩 Notes & Tips

* 🧠 **Axis order:** TIFF volumes are indexed as `[X, Y, Z]` / array order as read by the script, while Avizo coordinates are read from `.am` files and converted internally. Check coordinate conventions carefully if adapting the script.
* 📍 **Medialness matters:** off-centre graph points can bias the minimum-area reslice toward a longitudinal cut and artificially reduce measured radius.
* 🎯 **Sampling:** `sample_n` controls how many graph points are evaluated per file.
* 📏 **Subvolume size:** `subvolume_edge_size_cdm_factor` controls the local search region size. Larger values give more search context but increase runtime.
* ⏱️ **Runtime:** the multi-stage angular search can be computationally expensive for large numbers of sampled points.
* 🚫 **Boundary points:** points on the image boundary are skipped.

---
<br><br>
## 🧾 Output Example

The script prints one row per sampled point, including:

* sample number
* graph radius estimate
* measured effective radius
* local CDM value
* signed relative error
* absolute relative error

This makes it useful for:

* checking whether graph radius values systematically over- or under-estimate vessel radius
* comparing different skeletonization methods or radius correction methods
* identifying graph files with poor local radius fidelity

---
<br><br>
## 🧾 Citation

If you use this script in academic work, please cite your project and link to this repository.
