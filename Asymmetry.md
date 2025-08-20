
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


<img width="1385" height="1371" alt="image" src="https://github.com/user-attachments/assets/7a705b34-23cf-49a8-a8b5-4ba83fba06f5" />
<img width="1454" height="1431" alt="image" src="https://github.com/user-attachments/assets/dd2e6585-ac06-4bbc-8625-be0c8c400f4f" />
<img width="1001" height="968" alt="image" src="https://github.com/user-attachments/assets/dd5a154e-25a8-4b7e-942c-40901465b9b6" />
<img width="1001" height="968" alt="image" src="https://github.com/user-attachments/assets/780ca66c-ad91-46e8-9bc4-54c0bf4f29b8" />
<img width="2101" height="755" alt="image" src="https://github.com/user-attachments/assets/b2da9822-f22f-4f4e-b784-0b7eecdeb434" />




