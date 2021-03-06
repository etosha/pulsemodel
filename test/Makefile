# Run analysis/re-synthesis (copy-synthesis) on some samples
# 
# Description
#   * For the sake of completness this script is extracting a spectral envelope
#     (somestimes called spectrogram in vocoding) (named $FILENAME.spec below)
#     The used method is a SIMPLISTIC NASTY IDIOTIC Linear interpolation.
#     Thus, please replace this envelope by a better one.
#     (e.g. that of STRAIGHT vocoder)
#   * This script uses pre-computed f0 files for waveforms that are downloaded
#     from the CMU Arctic databases (http://festvox.org/cmu_arctic/cmu_arctic/).
#     If REAPER F0 estimator is installed (https://github.com/google/REAPER)
#     , you can also generate the f0 using the analysis.py script directly
#     (see below for the rule commented).
# 
# Usage
# 	$ make 
# 
# Copyright(C) 2016 Engineering Department, University of Cambridge, UK.
# 
# License
#    Licensed under the Apache License, Version 2.0 (the "License");
#    you may not use this file except in compliance with the License.
#    You may obtain a copy of the License at
# 
#      http://www.apache.org/licenses/LICENSE-2.0
# 
#    Unless required by applicable law or agreed to in writing, software
#    distributed under the License is distributed on an "AS IS" BASIS,
#    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
#    See the License for the specific language governing permissions and
#    limitations under the License.
# 
# Author
#     Gilles Degottex <gad27@cam.ac.uk>

.PRECIOUS: %.resynth.wav %.wav %.f0 %.spec %.pdd %.nm %.bndnm

RESYNTHESIS=slt_arctic_a0010.resynth.wav bdl_arctic_a0020.resynth.wav clb_arctic_a0030.resynth.wav awb_arctic_a0040.resynth.wav

all:
	$(MAKE) samples.scp
	$(MAKE) ${RESYNTHESIS}

samples.scp:
	wget http://festvox.org/cmu_arctic/cmu_arctic/cmu_us_slt_arctic/wav/arctic_a0010.wav -O slt_arctic_a0010.wav
	wget http://festvox.org/cmu_arctic/cmu_arctic/cmu_us_bdl_arctic/wav/arctic_a0020.wav -O bdl_arctic_a0020.wav
	wget http://festvox.org/cmu_arctic/cmu_arctic/cmu_us_clb_arctic/wav/arctic_a0030.wav -O clb_arctic_a0030.wav
	wget http://festvox.org/cmu_arctic/cmu_arctic/cmu_us_awb_arctic/wav/arctic_a0040.wav -O awb_arctic_a0040.wav
	ls *.wav |grep -v '.resynth.wav' > samples.scp


# Analysis ---------------------------------------------------------------------

%.f0 %.spec %.pdd: %.wav %.f0.txt ../analysis.py
	@python ../analysis.py $< --inf0txt $(<:.wav=.f0.txt) --f0 $(<:.wav=.f0) --spec $(<:.wav=.spec) --pdd $(<:.wav=.pdd)

# # You can use this rule if you want to skip PDD and use the noisemask
# %.f0 %.spec %.nm: %.wav %.f0.txt ../analysis.py
# 	python ../analysis.py $< --inf0txt $(<:.wav=.f0.txt) --f0 $(<:.wav=.f0) --spec $(<:.wav=.spec) --nm $(<:.wav=.nm)
# 
# # Or this rule if you also want to compress the noisemask in mel bands
# %.f0 %.spec %.bndnm: %.wav %.f0.txt ../analysis.py
# 	@python ../analysis.py $< --inf0txt $(<:.wav=.f0.txt) --f0 $(<:.wav=.f0) --spec $(<:.wav=.spec) --nm $(<:.wav=.bndnm) --nm_nbbnds 21

# # If you have REAPER F0 estimator installed, you can also generate the f0 directly:
# %.f0 %.spec %.pdd: %.wav ../analysis.py
# 	@python ../analysis.py $< --f0 $(<:.wav=.f0) --spec $(<:.wav=.spec) --pdd $(<:.wav=.pdd)


# Synthesis --------------------------------------------------------------------

%.resynth.wav: %.f0 %.spec %.pdd ../synthesis.py
	@python ../synthesis.py $@ --f0file $< --specfile $(<:.f0=.spec) --pddfile $(<:.f0=.pdd) --fs 16000

# # You can use this rule if you want to skip PDD and use the noisemask
# %.resynth.wav: %.f0 %.spec %.nm ../synthesis.py
# 	@python ../synthesis.py $@ --f0file $< --specfile $(<:.f0=.spec) --noisemaskfile $(<:.f0=.nm) --fs 16000

# # Or this rule if you also want to compress the noisemask in mel bands
# %.resynth.wav: %.f0 %.spec %.bndnm ../synthesis.py
# 	@python ../synthesis.py $@ --f0file $< --specfile $(<:.f0=.spec) --noisemaskfile $(<:.f0=.bndnm) --noisemask_nbbnds 21 --fs 16000

clean:
	rm -f *.resynth.wav *.f0 *.spec *.pdd *.nm *.bndnm *.diff

distclean:
	rm -f samples.scp *.wav *.f0 *.spec *.pdd *.nm *.bndnm *.diff

	
# For debugging purpose --------------------------------------------------------
.PHONY: testref makeref

%.resynth.diff: %.resynth.wav %.resynth.ref.wav
	cmp $^ > $@

testref: $(RESYNTHESIS:.resynth.wav=.resynth.diff)
	$(NOOP)

makeref:
	$(foreach f,$(RESYNTHESIS),cp -f $(f) $(f:.resynth.wav=.resynth.ref.wav);)
	rm -f $(RESYNTHESIS:.resynth.wav=.resynth.diff)
