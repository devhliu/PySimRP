# PySimRP Design Document

## Overview

PySimRP is a comprehensive Python framework designed for simulating radiopharmaceutical pharmacokinetics and pharmacodynamics (PKPD) integrated with anatomical modeling, imaging simulation, and image reconstruction. The framework provides a modular, extensible architecture that enables researchers and clinicians to develop, test, and validate computational models of radiopharmaceutical behavior across multiple scales—from molecular kinetics to whole-body distribution and imaging acquisition.

The primary objectives of PySimRP are:

- **Integrated Simulation**: Seamlessly combine PKPD/ADME (Absorption, Distribution, Metabolism, Excretion) modeling with anatomical representations for realistic radiopharmaceutical distribution prediction
- **Imaging Realism**: Simulate realistic imaging acquisition processes including detector response, noise, and physical imaging phenomena
- **Reconstruction Capability**: Implement image reconstruction algorithms to process simulated imaging data
- **Extensibility**: Provide a framework that allows developers to easily extend functionality for new radiopharmaceuticals, anatomical models, and imaging modalities
- **Validation**: Support comprehensive testing and validation of models against experimental and clinical data

## Architecture Overview

PySimRP follows a modular architecture organized into interconnected layers:

```
┌─────────────────────────────────────────────────────────────┐
│           User Interface & Pipeline Orchestration            │
│        (Workflow Definition, Parameter Configuration)        │
└──────────────────────┬──────────────────────────────────────┘
                       │
         ┌─────────────┼─────────────┐
         │             │             │
    ┌────▼────┐  ┌────▼────┐  ┌────▼────┐
    │ PKPD    │  │Anatomical│  │ Imaging │
    │ Module  │  │ Module   │  │ Module  │
    └────┬────┘  └────┬────┘  └────┬────┘
         │             │             │
         └─────────────┼─────────────┘
                       │
         ┌─────────────┼─────────────┐
         │             │             │
    ┌────▼──────┐  ┌──▼──────┐  ┌──▼──────────┐
    │ Imaging   │  │Data     │  │Extensibility│
    │ Pipeline  │  │Mgmt     │  │Framework    │
    └───────────┘  └─────────┘  └─────────────┘
         │
    ┌────▼─────────────────────────────────────┐
    │  Image Reconstruction Module              │
    └──────────────────────────────────────────┘
```

### Core Design Principles

1. **Modularity**: Each functional domain (PKPD, anatomy, imaging) is independently developed and tested
2. **Separation of Concerns**: Clear interfaces between modules reduce coupling and facilitate maintenance
3. **Data-Driven**: Emphasis on standardized data formats and structures for interoperability
4. **Extensibility**: Plugin architecture and inheritance-based extension mechanisms allow customization
5. **Composability**: Modules can be combined in various configurations to support different simulation scenarios

## PKPD/ADME Modeling Module

### Purpose

The PKPD/ADME module handles the pharmacokinetic and pharmacodynamic modeling of radiopharmaceuticals, including absorption, distribution, metabolism, and excretion processes. This module is responsible for calculating time-dependent activity concentrations in various tissue compartments.

### Key Components

#### Compartment Models
- **Multi-compartment systems**: Support for linear and non-linear compartment structures
- **First-pass metabolism**: Modeling of initial hepatic or other organ metabolism
- **Tissue-specific kinetics**: Different kinetic parameters for distinct anatomical regions
- **Binding dynamics**: Receptor/enzyme binding and dissociation kinetics

#### Physiological Parameters
- **Tissue volumes**: Anatomically-derived compartment volumes
- **Blood flow rates**: Organ and tissue perfusion parameters
- **Permeability coefficients**: Tissue uptake and clearance rates
- **Metabolism constants**: Rate constants for metabolism and elimination

#### Kinetic Solvers
- **Differential equation solvers**: Numerical solutions for compartment ODEs
- **Steady-state calculations**: Time-independent activity distributions for long-acquisition scenarios
- **Population kinetics**: Support for inter-individual variability through parameter distributions

### Dependencies for Extensions

Future extensions of the PKPD module depend on:
- **Anatomical Module**: For compartment volume and perfusion data derived from anatomical models
- **Data Management**: For storing and retrieving kinetic parameters and experimental measurements
- **Extensibility Framework**: For registering custom compartment models and solver algorithms

## Anatomical Modeling Module

### Purpose

The Anatomical Modeling Module provides representations of human anatomy for localizing radiopharmaceutical distribution. It integrates tissue and organ definitions, supports multiple anatomical representations (voxel-based, surface-based, hybrid), and provides spatial queries for activity localization.

### Key Components

#### Anatomical Representations
- **Voxel grids**: High-resolution 3D grid structures representing tissue types and density
- **Organ segmentation**: Labeled regions corresponding to anatomically distinct organs and tissues
- **Surface meshes**: Boundary representations for efficient spatial computation
- **Tissue hierarchy**: Organized tissue type relationships (organ → tissue → cellular compartment)

#### Spatial Utilities
- **Coordinate transformations**: Conversion between anatomical, imaging, and computational coordinate systems
- **Tissue identification**: Query functions to determine tissue type at given spatial locations
- **Volume calculations**: Computation of compartment volumes from anatomical representations
- **Distance metrics**: Spatial relationships between anatomical structures

#### Anatomical Data
- **Reference atlases**: Standard human anatomy templates
- **Organ-specific parameters**: Dimensions, volumes, and perfusion characteristics of organs
- **Variability models**: Support for anatomical variations across patient populations

### Dependencies for Extensions

Future extensions of the Anatomical Module depend on:
- **Data Management**: For storing anatomical models and tissue parameter databases
- **Extensibility Framework**: For registering custom anatomical representations and query algorithms
- **PKPD Module**: For mapping kinetic compartments to anatomical regions

## Imaging Module

### Purpose

The Imaging Module simulates the process of radiopharmaceutical imaging acquisition, including emission processes, detector response, physical phenomena (scatter, attenuation), and image formation. This module bridges the gap between theoretical activity distributions and realistic acquired images.

### Key Components

#### Emission Simulation
- **Decay processes**: Accurate modeling of radioactive decay, including decay schemes with branching ratios
- **Photon generation**: Simulation of primary emissions (annihilation photons in PET, single photons in SPECT)
- **Spatial distribution**: Positioning of decay events according to activity concentration distributions
- **Energy spectral simulation**: Energy-dependent photon generation for realistic spectra

#### Physical Phenomena
- **Attenuation**: Photon attenuation according to tissue composition and density
- **Scatter**: Compton scatter effects in detector materials and tissues
- **Dead time**: Detector dead time losses in high-count-rate scenarios
- **Geometric efficiency**: Detection probability based on detector geometry and configuration

#### Detector Response
- **Energy resolution**: Photon energy measurement uncertainty
- **Position resolution**: Spatial resolution of photon localization
- **Intrinsic detection efficiency**: Probability of photon detection within active detector volume
- **Timing resolution**: Time measurement uncertainty for coincidence detection (PET)

#### Acquisition Simulation
- **Projection generation**: Formation of projection images from 3D activity distribution
- **List-mode data**: Event-by-event simulation for advanced processing
- **Sinogram formation**: Reformatting of projections into sinogram representation
- **Acquisition protocols**: Flexibility to simulate various clinical imaging protocols

### Dependencies for Extensions

Future extensions of the Imaging Module depend on:
- **PKPD Module**: For activity concentration data at various time points
- **Anatomical Module**: For tissue composition and density for attenuation/scatter correction
- **Data Management**: For storing detector specifications and acquisition parameters
- **Reconstruction Module**: For downstream processing of simulated imaging data
- **Extensibility Framework**: For registering custom detector models and physics engines

## Reconstruction Module

### Purpose

The Reconstruction Module implements image reconstruction algorithms to convert simulated projection data (or raw detector data) into 3D image volumes suitable for visualization and analysis. This module supports both analytical and iterative reconstruction methods.

### Key Components

#### Analytical Reconstruction
- **Filtered backprojection**: FBP algorithm for rapid image reconstruction
- **Filter design**: Various filter kernels (Hann, Hamming, Butterworth) for noise-frequency tradeoff
- **Ramp filter**: Frequency-domain filtering for projection data
- **3D reconstruction variants**: Extension of FBP to 3D imaging geometries

#### Iterative Reconstruction
- **Statistical methods**: Maximum likelihood expectation maximization (MLEM), ordered subset EM (OSEM)
- **Penalty functions**: Incorporation of regularization terms (quadratic, total variation, edge-preserving)
- **Convergence criteria**: Stopping rules and iteration count management
- **Acceleration techniques**: Ordered subset approaches for computational efficiency

#### Correction Methods
- **Attenuation correction**: Incorporation of attenuation maps (typically from CT or transmission imaging)
- **Scatter correction**: Reduction of scatter-induced biases
- **Detector normalization**: Sensitivity correction for detector efficiency variations
- **Dead time correction**: Compensation for detector count rate losses

#### Output Processing
- **Volume representation**: 3D image array generation with appropriate spacing and orientation
- **Noise characterization**: Variance and resolution measurements
- **Quality metrics**: Quantitative assessment of reconstruction quality

### Dependencies for Extensions

Future extensions of the Reconstruction Module depend on:
- **Imaging Module**: For simulated projection data and detector specifications
- **Anatomical Module**: For attenuation and scatter correction maps
- **Data Management**: For storing reconstruction parameters and algorithms
- **Extensibility Framework**: For registering custom reconstruction algorithms and penalty functions

## Pipeline Integration

### Purpose

The Pipeline Integration layer provides orchestration of simulation workflows, coordinating execution across multiple modules and managing data flow between processing stages.

### Key Components

#### Workflow Definition
- **Pipeline specification**: Declarative definition of simulation workflows
- **Parameter specification**: Configuration of module-specific parameters
- **Execution order**: Automatic dependency resolution and scheduling
- **Progress monitoring**: Tracking of simulation progress and status reporting

#### Data Flow Management
- **Intermediate storage**: Temporary storage of data between pipeline stages
- **Format conversion**: Automatic conversion between module-specific data formats
- **Checkpointing**: Ability to restart from intermediate states for long-running simulations
- **Memory management**: Efficient handling of large 3D datasets during pipeline execution

#### Simulation Scenarios
- **Multi-timepoint simulations**: Sequential simulations at different time points post-injection
- **Multi-modality scenarios**: Coordinated simulation of multiple imaging modalities
- **Parameter sweep studies**: Systematic variation of parameters for sensitivity analysis
- **Population studies**: Batch processing of multiple patient/anatomical configurations

### Dependencies for Extensions

Pipeline Integration depends on:
- **PKPD Module**: For kinetic calculations
- **Anatomical Module**: For spatial information
- **Imaging Module**: For acquisition simulation
- **Reconstruction Module**: For image processing
- **Data Management**: For accessing and storing pipeline-related data
- **Extensibility Framework**: For discovering and registering pipeline stages

## Data Management

### Purpose

The Data Management system provides centralized handling of simulation data, including storage, retrieval, caching, and format specification. It ensures consistency across modules and facilitates reproducibility of simulations.

### Key Components

#### Data Storage
- **Structured storage**: Database and file-based storage of simulation data
- **Hierarchical organization**: Logical organization of simulations, experiments, and results
- **Metadata management**: Comprehensive tracking of simulation parameters and provenance
- **Archival support**: Long-term storage and versioning of simulations

#### Data Formats
- **Standard formats**: DICOM for medical imaging, HDF5 for large numerical datasets
- **Format conversion**: Utilities for converting between internal and standard formats
- **Schema definition**: Validation of data against defined schemas
- **Custom formats**: Support for domain-specific data representations

#### Configuration Management
- **Parameter databases**: Storage of standard parameters (anatomical, physical, kinetic)
- **Configuration profiles**: Pre-defined configurations for common simulation scenarios
- **Parameter versioning**: Tracking of parameter set changes over time
- **Lookup utilities**: Efficient retrieval of parameters by name, category, or anatomical region

#### Data Query and Access
- **Query interface**: Flexible querying of stored simulations and results
- **Result retrieval**: Access to intermediate and final simulation results
- **Caching**: Fast retrieval of frequently accessed data
- **Streaming support**: Efficient processing of large datasets without full memory loading

### Dependencies for Extensions

Data Management depends on:
- **Extensibility Framework**: For registering custom data formats and storage backends

Data Management serves as a dependency for:
- All other modules for storing and retrieving parameters and data

## Extensibility Framework

### Purpose

The Extensibility Framework provides mechanisms for developers to extend PySimRP with new models, algorithms, and capabilities without modifying core code. It supports plugin registration, dependency injection, and interface-based development.

### Key Components

#### Plugin Architecture
- **Module discovery**: Automatic discovery and loading of extension modules
- **Registration mechanism**: Standardized registration of custom implementations
- **Version management**: Compatibility checking between plugins and core framework
- **Dependency resolution**: Automatic resolution of plugin dependencies

#### Interface Definitions
- **Abstract base classes**: Clear interface definitions for extensible components
- **Method contracts**: Specification of required methods and return types
- **Validation**: Runtime validation of plugin implementations against interfaces
- **Documentation**: Clear documentation of extension points and requirements

#### Inheritance and Composition
- **Base class hierarchy**: Inheritance-based extension for common patterns
- **Mixin support**: Composable functionality through multiple inheritance
- **Delegation patterns**: Composition-based extensions for complex behavior
- **Adapter patterns**: Compatibility wrappers for external tools and libraries

#### Configuration and Setup
- **Configuration loaders**: Loading extension configurations from files
- **Environment variables**: Configuration through environment settings
- **Setup utilities**: Installation and validation of extensions
- **Testing support**: Tools for testing extension implementations

### Extensibility Points

Key extension points include:

1. **Custom PKPD Models**: Define new compartment structures and kinetic equations
2. **Custom Anatomical Representations**: Implement new ways to represent anatomy
3. **Custom Detectors and Emission Models**: Extend imaging simulation capabilities
4. **Custom Reconstruction Algorithms**: Implement new reconstruction methods and penalty functions
5. **Custom Data Formats**: Add support for new data storage formats
6. **Custom Pipeline Stages**: Insert new processing stages into simulation workflows
7. **Custom Analysis Tools**: Add post-simulation analysis capabilities

### Dependencies for Extensions

The Extensibility Framework has minimal dependencies on other modules. All modules depend on:
- **Extensibility Framework**: For registration and interface definition

## Technical Requirements

### System Requirements
- **Python version**: 3.8 or later for core functionality, with recommended 3.10+
- **Operating systems**: Linux, macOS, and Windows support
- **Memory**: Minimum 4 GB RAM for basic simulations; 16+ GB recommended for high-resolution anatomical models and 3D imaging data
- **Processor**: Multi-core processors recommended for iterative reconstruction algorithms

### Core Dependencies
- **Numerical computing**: NumPy for array operations and mathematical functions
- **Scientific computing**: SciPy for advanced numerical algorithms (ODE solvers, signal processing)
- **Visualization**: Matplotlib for 2D visualization; optional 3D visualization libraries (VTK, mayavi)
- **Data I/O**: Nibabel for medical image formats, h5py for HDF5 support
- **Logging and configuration**: Standard library logging with configuration files

### Optional Dependencies
- **Advanced visualization**: VTK or mayavi for 3D rendering
- **Parallel processing**: Numba for JIT compilation of compute-intensive functions
- **GPU acceleration**: CuPy or GPU-enabled frameworks for large-scale simulations
- **Clinical integration**: DICOM libraries for seamless medical imaging integration

### Code Organization
- **Module structure**: Clear separation of concerns with well-defined module boundaries
- **Import management**: Explicit, non-circular imports to ensure maintainability
- **Naming conventions**: PEP 8 compliant naming for classes, functions, and variables
- **Documentation**: Comprehensive docstrings for all public APIs

## Testing and Validation

### Testing Strategy

#### Unit Testing
- **Module isolation**: Each module tested independently with mocked dependencies
- **Interface validation**: Verification that implementations conform to defined interfaces
- **Edge cases**: Testing of boundary conditions and exceptional inputs
- **Regression testing**: Automated tests to prevent reintroduction of fixed bugs

#### Integration Testing
- **Module interactions**: Testing of data flow between modules
- **Pipeline execution**: End-to-end testing of complete simulation workflows
- **Data consistency**: Verification of data integrity across pipeline stages
- **Performance benchmarking**: Monitoring of execution time and memory usage

#### Validation Studies
- **Analytical benchmarks**: Comparison against analytical solutions for simple cases
- **Synthetic data validation**: Testing against simulated data with known properties
- **Clinical correlation**: Comparison with experimental and clinical imaging data when available
- **Sensitivity analysis**: Assessment of how changes in parameters affect results

#### Test Coverage
- **Code coverage**: Automated measurement of test code coverage, targeting >80% for core modules
- **Critical path coverage**: 100% coverage for critical computation paths
- **Error handling**: Testing of error conditions and exception handling
- **Documentation examples**: Validation that documented examples execute correctly

### Validation Metrics
- **Kinetic accuracy**: Comparison of simulated kinetics with reference literature values
- **Spatial accuracy**: Positional accuracy of simulated activity distributions
- **Image quality metrics**: Quantitative assessment of reconstructed image quality (noise, resolution, bias)
- **Computational accuracy**: Numerical accuracy of reconstruction algorithms against reference implementations

## Documentation and Examples

### Documentation Structure

#### API Documentation
- **Module-level documentation**: Overview and purpose of each module
- **Class documentation**: Comprehensive docstrings with attributes and methods
- **Function documentation**: Parameter descriptions, return types, and examples
- **Type hints**: Full type annotations for all public APIs

#### User Guides
- **Getting started**: Installation and basic usage tutorial
- **Module guides**: Detailed guides for using each major module
- **Pipeline definition**: Instructions for creating custom simulation workflows
- **Parameter specification**: Guide to available parameters and their meanings
- **Troubleshooting**: Common issues and their solutions

#### Developer Guides
- **Architecture overview**: Detailed explanation of module interactions
- **Extension guide**: Step-by-step instructions for creating extensions
- **Testing guide**: Instructions for writing and running tests
- **Contributing guide**: Contribution guidelines and development workflow

#### Reference Documentation
- **Parameter reference**: Complete listing of available parameters with units and ranges
- **Algorithm reference**: Description of implemented algorithms with relevant citations
- **File format specifications**: Detailed specification of input/output file formats
- **API reference**: Complete API reference auto-generated from docstrings

### Example Simulations

#### Basic Examples
- **Simple kinetics**: Single-tissue model simulation with activity over time
- **Multi-organ distribution**: Simulation of radiopharmaceutical distribution across multiple organs
- **Basic imaging**: Simulation of PET or SPECT acquisition for standard protocols
- **Simple reconstruction**: Application of FBP reconstruction to simulated data

#### Advanced Examples
- **Multi-compartment kinetics**: Complex pharmacokinetic model with feedback loops
- **Population variability**: Simulation across anatomically or kinetically diverse populations
- **Iterative reconstruction**: Application of OSEM with attenuation and scatter correction
- **Parameter sensitivity**: Systematic study of how parameter changes affect imaging outcomes
- **Clinical protocol simulation**: Realistic simulation of standard clinical imaging protocols

#### Domain-Specific Examples
- **Oncology applications**: Tumor uptake and kinetics with therapeutic modeling
- **Cardiology applications**: Myocardial perfusion imaging simulation
- **Neurology applications**: Brain imaging with regional kinetic heterogeneity
- **Infection imaging**: Inflammatory marker distribution modeling

#### Integration Examples
- **Complete workflow**: End-to-end simulation from patient anatomy to reconstructed image
- **Batch processing**: Processing of multiple patients or time points
- **Comparison studies**: Comparative simulation of different radiopharmaceuticals
- **Validation studies**: Comparison of simulated and experimental data

---

**Document Version**: 1.0
**Last Updated**: December 26, 2025
**Status**: Active Design Document
