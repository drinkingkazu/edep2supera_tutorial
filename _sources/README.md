---
jupytext:
  cell_metadata_filter: -all
  formats: md:myst
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.10.3
kernelspec:
  display_name: Python 3
  language: python
  name: python3
---

# Getting started
This note covers a brief overview of running `ParticleBomb`  event generator ([repository](https://github.com/DeepLearnPhysics/EventGenerator)), which is used for generating simulation samples to train data reconstruction models in `lartpc_mlreco3d` package ([repository](https://github.com/DeepLearnPhysics/lartpc_mlreco3d)). 

## What is an event generator?

In case you are not familiar with the term _event generator_, in High Energy Physics (HEP) experiments, an event generator is the first step of a simulation chain which consists of 3 major steps.

1. **Event Generator** ... generates a list of particles that will interact with an experiment detector. There are many event generators out there (e.g. GENIE, GLOBUS, GIBU, ...) but they all do the same: generate a list of particles.
  
2. **Particle Tracking** ... takes the output of 1 (i.e. a list of particles), and simulate how each particle interacts with the detector medium. For this stage, almost all HEP experiments use a software called Geant4, which contains detailed physics models of how all kinds of particles interact with all kinds of materials. The output of Geant4 is energy deposition information in the form of a particle trajectory. For instance, if your event generator provide 1 muon as an input to this stage of simulation, Geant4 simulates how this muon go through all kinds of micro-physics processes such as multiple coulomb scattering, radiation of another particle, an interaction with electromagnetic field, a decay, energy deposition along its trajectory in a medium, etc.. 

3. **Detector response** ... takes the energy deposition information from Geant4, and simulates how your detector respond to them. This may include how your detector signal is created (e.g. for LArTPC, this might be ionization electrons and scintillation photons), how those signal propagate to sensitive detector electronics, and how those electronics respond to the signal. The output of this stage mimics the data that is recorded by your detector.
