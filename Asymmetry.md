
We have the FreeSurfer ASL perfusion Statistical data files (CSV, JSON), aiming to examine the format of the stats files to calculate the asymmetry analyses.

```
# Segmentation Statistics 
  ColHeaders Index SegId NVoxels Volume_mm3 StructName Mean StdDev Min Max Range
  1  10    1234  1234.0  Left-Thalamus-Proper      45.2   8.1  25.0  65.0  40.0
  2  49    1289  1289.0  Right-Thalamus-Proper     47.8   8.4  26.0  68.0  42.0
```

### Required Columns

- **Index**: Sequential row number
- **SegId**: FreeSurfer segmentation ID
- **NVoxels**: Number of voxels in the region
- **Volume_mm3**: Region volume in cubic millimeters
- **StructName**: Anatomical structure name (must include "Left-" and "Right-" prefixes)
- **Mean**: Mean perfusion value (primary analysis target)
- **StdDev**: Standard deviation of perfusion values
- **Min/Max/Range**: Additional statistics

### 1. Laterality Index (LI)
**Formula**: `(L - R) / (L + R)`
- Range: -1 to +1
- Positive values indicate left > right

### 2. Asymmetry Index (AI)
**Formula**: `(L - R) / ((L + R) / 2) × 100`
- Clinically interpretable threshold: AI < x indicates hypo/hyper perfusion


### Project Structure

```
AsymmetryAnalysis/
├── README.md                    # This file
├── requirements.txt             # Python dependencies
├── src/                        # Source code
│   ├── __init__.py
│   ├── parser.py               # FreeSurfer data parsing
│   ├── calculator.py           # Asymmetry calculations
│   ├── statistics.py           # Statistical analysis
│   ├── visualizer.py           # Visualization methods
│   ├── exporter.py             # Data export utilities
│   └── analysis.py             # Main analysis pipeline
├── examples/                   # Usage examples
│   └── run_analysis.py         # Complete analysis example
├── docs/                       # Documentation
│   ├── user_guide.md          # User guide
│   └── clinical_applications.md # Clinical applications
└── tests/                     # Test files
    └── test_asymmetry.py      # Unit tests
```


================================================================================
BRAIN ASYMMETRY ANALYSIS REPORT
================================================================================
Number of Regions: 14

SUMMARY STATISTICS
----------------------------------------
Mean Laterality Index: -0.0846
Standard Deviation: 0.1764
Range: -0.5523 to 0.0786
Most Asymmetric Region: Inf-Lat-Vent (LI = -0.5523)

DETAILED REGIONAL RESULTS
----------------------------------------
```
Region: Cerebral-White-Matter
  Left Perfusion: 22.76 ± 20.60
  Right Perfusion: 25.55 ± 23.96
  Laterality Index: -0.0578
  Asymmetry Index: -11.55%

Region: Lateral-Ventricle
  Left Perfusion: 22.37 ± 23.80
  Right Perfusion: 50.35 ± 49.52
  Laterality Index: -0.3847
  Asymmetry Index: -76.93%

Region: Inf-Lat-Vent
  Left Perfusion: 12.69 ± 13.39
  Right Perfusion: 43.99 ± 48.08
  Laterality Index: -0.5523
  Asymmetry Index: -110.45%

Region: Cerebellum-White-Matter
  Left Perfusion: 34.03 ± 25.36
  Right Perfusion: 42.88 ± 37.97
  Laterality Index: -0.1151
  Asymmetry Index: -23.01%

Region: Cerebellum-Cortex
  Left Perfusion: 52.73 ± 42.46
  Right Perfusion: 61.76 ± 48.37
  Laterality Index: -0.0789
  Asymmetry Index: -15.78%

Region: Thalamus-Proper
  Left Perfusion: 55.29 ± 63.56
  Right Perfusion: 54.04 ± 38.30
  Laterality Index: 0.0114
  Asymmetry Index: 2.29%

Region: Caudate
  Left Perfusion: 40.53 ± 24.41
  Right Perfusion: 45.37 ± 19.23
  Laterality Index: -0.0563
  Asymmetry Index: -11.26%

Region: Putamen
  Left Perfusion: 54.92 ± 25.84
  Right Perfusion: 52.98 ± 25.06
  Laterality Index: 0.0180
  Asymmetry Index: 3.60%

Region: Pallidum
  Left Perfusion: 39.55 ± 21.30
  Right Perfusion: 35.49 ± 23.47
  Laterality Index: 0.0541
  Asymmetry Index: 10.82%

Region: Hippocampus
  Left Perfusion: 53.25 ± 31.76
  Right Perfusion: 60.77 ± 48.75
  Laterality Index: -0.0659
  Asymmetry Index: -13.18%

Region: Amygdala
  Left Perfusion: 48.33 ± 32.92
  Right Perfusion: 41.29 ± 40.37
  Laterality Index: 0.0786
  Asymmetry Index: 15.71%

Region: Accumbens-area
  Left Perfusion: 54.85 ± 25.05
  Right Perfusion: 59.16 ± 25.90
  Laterality Index: -0.0378
  Asymmetry Index: -7.56%

Region: VentralDC
  Left Perfusion: 48.37 ± 33.27
  Right Perfusion: 55.85 ± 49.01
  Laterality Index: -0.0718
  Asymmetry Index: -14.36%

Region: choroid-plexus
  Left Perfusion: 87.51 ± 45.03
  Right Perfusion: 75.57 ± 28.55
  Laterality Index: 0.0732
  Asymmetry Index: 14.65%
```

  <img width="1315" height="886" alt="image" src="https://github.com/user-attachments/assets/879101a5-c648-4cfa-99d8-cd2d1c02fba5" />

