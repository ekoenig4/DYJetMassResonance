# DYJet Mass Resonance

This repo walks through a general implementation of finding a Z mass peak using Drell-Yan (DY) + Jets Monte Carlo (MC) samples. 

## DY Jet MC
Drell-Yan is a process where a Z boson decays to two oppositely charged leptons. In these MC samples, this process is simulated with decays to electrons, muons, and taus. The MC samples are split into different total transverse energy (HT) bins, each with their own cross section. 

## Mass Resonance
It is possible to reconstruct the mass resonance of the Z boson (~ 91 GeV) using the four vectors of the two leptons from the decay. This can be implemented in pyroot as follows

```python
import ROOT as rt
lep1_p4 = rt.Math.PtEtaPhiE(lep1_pt,lep1_eta,lep1_phi,lep1_E) # First lepton kinematic information
lep2_p4 = rt.Math.PtEtaPhiE(lep2_pt,lep2_eta,lep2_phi,lep2_E) # Second lepton kinematic information
dilep_p4 = lep1_p4 + lep2_p4 # Add the two lepton four-vectors to make the di-lepton four-vector
dilep_mass = dilep_p4.M() # Get the mass of the di-lepton system (Z Mass)
```

## The MC Files
You can download a tar.gz file that has the HT binned DYJets sample off of HiperGator. If you don't have a HiperGator account, this [tutorial](https://github.com/rosedj1/CMSOfficeHours/blob/master/UF/HiPerGator.md#whose-group-are-you-in). You can download the tar.gz file to your local computer as follows

```bash
scp username@hpg2.rc.ufl.edu:/home/ekoenig/public/dyjet_ht_2018_ntuples.tar.gz .
tar -xvf dyjet_ht_2018_ntuples.tar.gz
```

Each of these files have a TTree called eventTree. You can open one file in a TBrowser 

```bash
root -l dyjet_ht_2018_ntuples/MC_DYJetsToLL_HT600-800_1.root -e "new TBrowser"
```

or if you have the command, rootbrowse, you can use

```bash
rootbrowse dyjet_ht_2018_ntuples/MC_DYJetsToLL_HT600-800_1.root
```

Mose installations of root have this command. Openning the TTree you can see that there are branches for a bunch of different objects for each event, the ones we will be intereseted in first is
1. nEle: Number of electrons in the event
2. elePt: List of Pt for each electron in the event
3. eleEta: List of eta for each electron in the event
4. elePhi: List of phi for each electron in the event
5. eleE: List of energy for each electron in the event
6. eleCharge: List of charge for each electron in the event

## 1st Step
The first step is to make the Z mass resonance using the electron information. I would recommend starting with just one of the MC files. You can make a simple event loop 

```python
import ROOT as rt
tfile = rt.TFile("dyjet_ht_2018_ntuples/MC_DYJetsToLL_HT600-800_1.root")
ttree= tfile.Get("eventTree")
for event in ttree:
  ... 
```

In the event loop, you should loop over each electron and find the first two with opposite charges. The branch info can be accessed as a field of the event like so

```python
event.nEle
event.elePt
```

Calculate the di-lepton mass and fill a histogram. I would recommend a histogram with 50 bins with a range of (60,120). 

## 2nd Step
Once you have made a di-electron mass histogram, you can add the muon pairs next. This is done the same way as the electrons, but this time looking at the muon information. 

## 3rd Step
Once you have both the di-electron and di-muon pairs you can move on to running the rest of the MC samples. You should keep the histograms separate for each file, because we have to stitch all the HT bins together to get the full DYJets distribution. Each HT bin has a corresponding cross section which tells us how many events of that process we should expect in real data. To be able to add the distributions of the HT bins together we need to calculate the scale factor for each bin. The scale factor is defined as

```
scale = xsec * (number of events passed)/(total number of events)
```

This scale tells us what fraction of events pass in real data. We can easly scale a TH1 histogram using the method

```python
hist.Scale(scale)
```

Once you have scaled all the histograms in this, we can now add all the histograms together. This can be done with the TH1 method

```python
hist_1.Add(hist_2)
```

Note that this method is actually adding hist_2 to hist_1, not giving you the addition of hist_1+hist_2. The cross sections for each HT bin is given in the dyjets_xsec.py file. Cross sections are given in pb. After doing this scaling, we can multiple these distributions by the total luminosity of the data we are studying to get the predicted number of events in the data. 
