## Pipelines for NGS population genetics -- mapgd computation in parallel
### Lynch Lab, CME, Biodesign, ASU 
#### Curated and updated by Xiaolong Wang <ouqd@hotmail.com>
#### Initialized May 28, 2018
		==============================================================
		#	How to make and run the parallel mapgd pipeline  #
		==============================================================
		
1. After mapping all the reads to the reference genome, you have got a number of .mpileup files: 

	========================================================
		SampleID-001.mpileup
		SampleID-002.mpileup
				...
		SampleID-096.mpileup
	========================================================

2. Make proview files: 
	========================================================
	
		perl MPMP.pl
		
	========================================================
	
	This will produce a parallel mapgd pipeline (mapgd-parallel.pbs).

3. Submit the parallel mapgd pipeline:
	===============================================

		qsub mapgd-parallel.pbs
		
	===============================================
	
This parallel mapgd pipeline will produce mapgd proview files in parallel and combine all mapgd proview files into one using 
a java program (CombineProview.java), and then do the rest of the mapgd pipeline same as the original mapgd pipeline.

Note: 

In the original pipeline,  mapgd proview files is produced by the following command:
	=========================================================================
 
		mapgd proview -i *.mpileup -H $HeaderFile > output.pro.txt 
		
 	=========================================================================

This is simple and straight forward. However, it is very slow because it is not fully parallelized. 
This step  will takes up to 50-100 hours for a population with 96 clones. 
To reduce the computation time, in this new pipeline (mapgd-parallel.pbs, produced by MPMP.pl), 
the proview files are generated for each of the 96 clones independently:
 
	=========================================================================
	mapgd proview -i $SampleID-001.mpileup -H $HeaderFile > $SampleID-001.proview &
	mapgd proview -i $SampleID-002.mpileup -H $HeaderFile > $SampleID-002.proview &
	mapgd proview -i $SampleID-003.mpileup -H $HeaderFile > $SampleID-003.proview &
	
			... ...
			
	mapgd proview -i $SampleID-096.mpileup -H $HeaderFile > $SampleID-096.proview &
	
	wait
	
	=========================================================================

In this way, mapgd proview file is produced for each of the 96 clones.  Because all processes can be run simutaneously in independent
threads, the computation time is greatly deduced. It also helps to identify a bad mpileup file more conviniently.

Then, the produced mapgd proview files are combined by using a homemade java program (CombineProview.java), 
which will find the mapgd proview files are combined them into one automatically:
	=============================================================
	
		java -cp ./ CombineProview <DATA_DIR> <output>
		
	=============================================================
	
		
======================END=======================================
