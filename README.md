# CImakeshift

[![Build Status](https://dev.azure.com/moritzbeutel/CImakeshift/_apis/build/status/mbeutel.CImakeshift?branchName=master)](https://dev.azure.com/moritzbeutel/CImakeshift/_build/latest?definitionId=7&branchName=master)

*CImakeshift* is a collection of templates for continuous integration (CI). Currently, it contains a template for continuous
integration testing of C++ code with [Azure Pipelines](https://azure.microsoft.com/en-us/products/devops/pipelines/) using
[Vcpkg](https://vcpkg.io/) for dependency management.

This template supports compilation of C and C++ code with MSVC, GCC, Clang, and CUDA code with NVCC.
(Note that CUDA code can only be compiled, not executed, when running on Microsoft-hosted agents because they do not come with
access to CUDA hardware.)
For a detailed list of the platforms and compiler versions supported, please refer to
[the comments in azure-pipelines/cmake.yml](azure-pipelines/cmake.yml).

This code is available under the [Boost Software License](LICENSE.txt), so feel free to reuse it. However, please note that
there is no documentation except for some comments in the code, and I cannot offer any guarantees of stability or compatibility
for the code in this repository, which I consider an implementation detail of my other public projects. If you would like to use
it, it is probably best to fork this repository and have your CI runners take a dependency on your own fork.

That being said, here is an annotated example, taken from [this repository's own CI configuration](ci/azure-pipelines.yml),
of a CI configuration with Azure Pipelines and the CImakeshift template:

```yaml
variables:
  # enable this to see debug output
  System.Debug: true

# select the branches for which a CI run is triggered
trigger:
- master
- mr/*

# select the branches for which a pull request triggers a CI run
pr:
- master

resources:
  repositories:
    - repository: CImakeshift
      type: github
      name: mbeutel/CImakeshift  # TODO: please substitute your own fork here
      endpoint: mbeutel

jobs:
- template: azure-pipelines/cmake.yml@CImakeshift
  parameters:
    caption: Basic
    vcpkgRef: 41c447cc210dc39aa85d4a5f58b4a1b9e573b3dc
    
    # Installation of packages in Vcpkg's Classic mode
    vcpkgPackages: ['gsl-lite']

    # optional, defaults to '$(Build.SourcesDirectory)'
    cmakeSourceDir: '$(Build.SourcesDirectory)/ci/basic'
    
    # explicit specification of CMake configuration args
    cmakeConfigArgs: '-DBUILD_TESTING=ON -DCMAKESHIFT_WARNING_LEVEL=high'

    targets:

    - os: Windows
      cxxCompiler: MSVC
      cxxCompilerVersions: [VS2019, VS2022]
      platforms: [x86, x64]

    - os: Windows
      cxxCompiler: Clang
      cxxCompilerVersions: [VS2019, VS2022]
      platforms: [x86, x64]

    - os: Windows
      cxxCompiler: MSVC
      cxxCompilerVersions: [VS2022]
      cudaCompiler: NVCC
      cudaCompilerVersions: [12_8]
      platforms: [x64]
      vcpkgPackages: ['<vcpkgPackages>', 'catch2']  # refer to job-wide packages specification with '<vcpkgPackages>'
      cudaPackages: ['nvrtc-dev']
      cmakeConfigArgs: '<cmakeConfigArgs> -DBUILD_TESTING_CUDA=ON'  # refer to job-wide config args with '<cmakeConfigArgs>'

    - os: Linux
      cxxCompiler: GCC
      cxxCompilerVersions: [9, 12]
      platforms: [x64]

    - os: Linux
      cxxCompiler: GCC
      cxxCompilerVersions: [14]
      cxxStandardLibraryDebugMode: true
      cxxSanitizers: [AddressSanitizer, UndefinedBehaviorSanitizer]
      platforms: [x64]
      tag: 'sanitize'
      cmakeBuildConfigurations: [Debug, RelWithDebInfo]

    - os: Linux
      cxxCompiler: Clang
      cxxCompilerVersions: [19]
      cxxStandardLibrary: libstdc++
      cxxStandardLibraryDebugMode: true
      cxxSanitizers: [AddressSanitizer, UndefinedBehaviorSanitizer, ImplicitIntegerArithmeticValueChange]
      platforms: [x64]
      tag: 'sanitize'
      vcpkgTriplet: <platform>-<os>
      cmakeBuildConfigurations: [Debug, RelWithDebInfo]

    - os: Linux
      cxxCompiler: GCC
      cxxCompilerVersions: [14]
      cudaCompiler: NVCC
      cudaCompilerVersions: [12_8]
      platforms: [x64]
      cudaPackages: ['<cudaPackages>', 'nvrtc-dev']
      cmakeConfigArgs: '<cmakeConfigArgs> -DBUILD_TESTING_CUDA=ON'

    - os: MacOS
      cxxCompiler: GCC
      cxxCompilerVersions: [11, 12, 13, 14]
      platforms: [x64]

    - os: MacOS
      cxxCompiler: AppleClang
      cxxCompilerVersions: [14, 15]
      platforms: [x64]

- template: azure-pipelines/cmake.yml@CImakeshift
  parameters:
    caption: Vcpkg manifests
    vcpkgRef: 41c447cc210dc39aa85d4a5f58b4a1b9e573b3dc

    cmakeSourceDir: '$(Build.SourcesDirectory)/ci/vcpkg-manifest'

    # optional, defaults to CMake source directory
    # (Vcpkg is operated in Manifest mode if a manifest is found, and in Classic mode otherwise)
    vcpkgManifestRoot: '$(Build.SourcesDirectory)/ci/vcpkg-manifest/root'

    targets:

    - os: Windows
      cxxCompiler: MSVC
      cxxCompilerVersions: [2022]
      platforms: [x64]
      cmakeConfigPreset: 'ci-msvc'  # refers to configuration preset in CMakePresets.json

    - os: Windows
      cxxCompiler: Clang
      cxxCompilerVersions: [VS2022]
      platforms: [x64]
      cmakeConfigPreset: 'ci-clang-cl'

    - os: Linux
      cxxCompiler: GCC
      cxxCompilerVersions: [13]
      platforms: [x64]
      cmakeConfigPreset: 'ci-gcc'

    - os: Linux
      cxxCompiler: Clang
      cxxCompilerVersions: [19]
      platforms: [x64]
      cmakeConfigPreset: 'ci-clang'

    - os: MacOS
      cxxCompiler: GCC
      cxxCompilerVersions: [13]
      platforms: [x64]
      cmakeConfigPreset: 'ci-gcc'

    - os: MacOS
      cxxCompiler: AppleClang
      cxxCompilerVersions: [14_0_3, 15, 16]
      platforms: [x64]
      cmakeConfigPreset: 'ci-clang'
```

For more examples, check out [some other libraries](https://github.com/gsl-lite/gsl-lite)
[which make use of](https://github.com/mbeutel/makeshift) [this template](https://github.com/mbeutel/patton).
