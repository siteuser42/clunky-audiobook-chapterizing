## m4b-mp3-audiobook-chapters-from-cuesheets
- A clunky but mostly automated way to merge audiobooks without an encoding step (remuxing only) to either single file m4b files with quicktime/nero chapters or single file mp3s with id3v2 chapters using cuesheets
	- [Good breakdown of different chapter types](https://github.com/Zeugma440/atldotnet/wiki/Focus-on-Chapter-metadata)

This is cobbled together from various scripts and software from github, googling, etc, but the majority is in .bat files so Windows is required (probably would be straightforward to convert to python/ruby except for the executables)

#### Full credit to creators of borrowed scripts/software (included in repository, mainly for redundancy, will remove if asked):
- [TestToCue.exe](https://community.mp3tag.de/t/generate-cue-file-from-tracklist/11750/13)
- [mergechapters.py](https://gist.github.com/cliss/53136b2c69526eeed561a5517b23cefa) (small edit made to remove extraneous video streams from embedded covers)
- [mp4_to_m4b.bat + mp4chaps](http://pds16.egloos.com/pds/200910/19/90/mp4_to_m4b.zip) (the .exe originates from MP4v2)
- [cue2ffmeta.rb](https://gist.github.com/remko/e15c4fe26d479e134f36#file-cue2ffmeta-rb)

#### Not everything that is needed is included in this repository! You will also need:
- [ffmpeg (tested with 4.4)](https://git.ffmpeg.org/ffmpeg.git)
- python3
- Ruby
- [rubycue](https://rubygems.org/gems/rubycue/versions/0.1.0) 
	- An edited cuesheet.rb is provided, which allows for 16+ hour books and turns off validation parsing, it should be placed in your-ruby-install-location\Ruby30-x64\lib\ruby\gems\3.0.0\gems\rubycue-0.1.0\lib\rubycue"
- [mp3directcut](https://mpesch3.de/) (v1.99 tested)
- [mp3tag](https://www.mp3tag.de/en/download.html)
- [rxrepl](https://sites.google.com/site/regexreplace/) (optional, checks for formatting errors but the .bat still works if it isn't present)
- findstr (should be pre-installed on windows, if cmd doesn't recognize it you can find it in system32 and can copy it into working directory)

#### General work flow is:
1. Merge multiple audio files to a single mp3/mp4 file
2. Obtain a .cue with desired chapter info (via a tracklist from mp3tag or pause detection by mp3directcut, I recommend using macros on Notepad++ to quickly edit chapter titles)
3. Convert .cue to ffmetadata
4. Combine ffmetadata with mp3/mp4 file

### Installation
- Install the above required programs separately
- Place all the .bat files and cue2ffmeta.rb and mergechapters.py in the bin folder of ffmpeg (as well as rxrepl and findstr if needed), this is your working directory
- Ensure cuesheet.rb in your rubycue installation was replaced with the copy provided
- Go to the desired workflow below and follow its steps

#### Working files should be called:
- input.mp3/.mp4 (this is generated by respective merge_ext.bat)
- cuesheet.cue (this is generated by text2cue.bat or how you should name cue after saving from mp3directcut)
- metadata.txt (this is generated by cue2metadata.bat)
- output is temporarily called output.mp3/mp4 then renamed to the album metadata field and moved to folder of same name
	- note if album field is blank it will be renamed to just ".mp3" - might try and fix this later
- Temporary files are created along the way as well, but should be deleted at the end of their associated batch proccess
- cleanup.bat will delete all working files (including input, excluding output) 
	- you may want to backup your cuesheet before running this!

### Creating single M4B file with embedded chapters from individual chapterized m4b files
	0. Note that [AudioBookConverter](https://github.com/yermak/AudioBookConverter) is a good program that can handle this scenario pretty well, so check that out if you want a GUI, can be buggy at times
	1. Add m4b files to bin folder of ffmpeg (4.4 plus)
	2. Import tracks into mp3tag
	3. Edit "title" fields as desired (full chapter names will take longer than simple Chapter 01) and ensure "track" fields are filled with integers (can use mp3tag > tools > auto number wizard)
	4. select all, right click > export > txt_taglist > Okay to create list.txt file
		- If this is first time, you will need to first edit "txt_taglist" and replace current text with below and save changes
			$filename(txt,utf-8)$loop(%_path%)%track%.%artist% - %title% - $div(%_length_seconds%,60)':'$num($mod(%_length_seconds%,60),2)
			$loopend()
	5. Use text2cue.bat or drag generated .txt file onto TextToCue.exe file, which generates a .cue file
		# if you manually do it, ensure the the cue has a TITLE field before next step
	6. Run merge_m4b.bat to merge m4b files to a single input.mp4 file (make sure tracks are in order by filename)
	7. Run cue2metadata.bat and merge_inputmp4_with_metadata.bat
	8. Run mp4tom4b.bat (converts fmpeg embedded chapters to proper m4b quicktime format), will also move output into its own folder and rename based on title field

### Creating single MP3 file with embedded chapters from individual chapterized mp3 files
	1. Add mp3 files to bin folder of ffmpeg (4.4 plus)
	2. Import tracks into mp3tag
	3. Edit "title" fields as desired and ensure "track" fields are filled  in format with integers (i.e. 01 or 1, not "1 of 42" etc.) (can use mp3tag > tools > auto number wizard to autofill track numbers)
	4. select all, right click > export > txt_taglist > Okay to create list.txt file
		- If this is first time, you will need to first edit "txt_taglist" and replace current text with below and save changes
			$filename(txt,utf-8)$loop(%_path%)%track%.%artist% - %title% - $div(%_length_seconds%,60)':'$num($mod(%_length_seconds%,60),2)
			$loopend()
	5. Use text2cue.bat or drag generated .txt file onto TextToCue.exe file, which generates a .cue file
			# if you manually do it, ensure the the cue has a TITLE field before next step
	6. Run merge_mp3.bat to merge mp3 files to a single input.mp3 file (make sure tracks are in order by filename)
	7. Run cue2metadata.bat and merge_inputmp3_with_metadata.bat, will also move output into its own folder and rename based on title field
		
### Creating single MP3 file with embedded chapters from randomly split mp3 files
	1. Add mp3 files to bin folder of ffmpeg (4.4 plus)
	2. Run merge_mp3.bat to merge mp3 files to a single input.mp3 file (make sure tracks are in order by filename)
	3. Drag merged mp3 file into mp3directcut
	4. Go to special > pause detection, try -44.5 dB, 2.9 s, 10 frames as a starting point
	5. Wait for it to detect chapter breaks, click close once it no longer says "stop"
	6. Check detected chapters with the >| dotted line button
		# If undercounts chapters, repeat step 4 with lower seconds or manually find chapters to add (c creates a chapter but only if correctly selected, quickly make chapters at cursor by hitting b, n, c)
		# If it overcounts by a lot, repeat step 4 with higher seconds
		# If only a few extra, can delete by highlighting with cursor (on the upper waveform not the rectangles) and go to special > remove edit break
		# Also can cross references number of chapters with numbers of pauses using epub
	7. File > Save as cuesheet.cue
	8. Open up .cue file in text editor and change TITLES as desired for each chapter (if you haven't modified rubycue script to turn off validation, add PERFORMER "Author Name" as second line)
	9. Run cue2metadata.bat and merge_inputmp3_with_metadata.bat, will also move output into its own folder and rename based on title field
	
### Creating single M4B file with embedded chapters from randomly split m4b files
	0. This one is the annoying type, since mp3directcut doesn't support m4b we will have to make a temporary reencode of the file to generate the cue sheet
	1. Add m4b files to bin folder of ffmpeg (4.4 plus)
	2. Run merge_m4b.bat to merge m4b files to a single input.mp4 file (make sure tracks are in order by filename)
	3. Open up a cmd terminal in the current directory and type:
	> ffmpeg -i input.mp4 output.mp3
	3. Drag reencoded mp3 file into mp3directcut
	4. Go to special > pause detection
	# Try -44.5 dB, 2.9 s, 10 frames
	5. Wait for it to detect chapter breaks, click close once it no longer says "stop"
	6. Check detected chapters with the >| dotted line button
		# If undercounts chapters, repeat step 4 with lower seconds or manually find chapters to add (c creates a chapter but only if correctly selected, quickly make chapters at cursor by hitting b, n, c)
		# If it overcounts by a lot, repeat step 4 with higher seconds
		# If only a few extra, can delete by highlighting with cursor (on the upper waveform not the rectangles) and go to special > remove edit break
		# Also can cross references number of chapters with numbers of pauses using epub
	7. File > Save as cuesheet.cue
	8. Open up .cue file in text editor and change TITLES as desired for each chapter (if you haven't modified rubycue script to turn off validation, add PERFORMER "Author Name" as second line)
	9. Once satisfied with your .cue, delete reencoded output.mp4
	10. Run cue2metadata.bat and merge_inputmp4_with_metadata.bat, will also move output into its own folder and rename based on title field
	
### Creating single M4B file that retains chapers from two m4b files with embedded chapters (can repeat if more than two files)
	1. change the names of the two m4b files to be merged to input1.mp4 and input2.mp4
	2. Run merge2m4b.bat, which will merge the two files, remove any video streams (from embedded covers) then run mp4_to_m4b.bat
	3. Can check with ffmpeg -i *.m4b or MediaInfo to ensure chapters are present
	4. If there is more than two m4b files, repeat steps for each additional part

	Optional manual use:
	1. call script with
		> py ./mergechapters.py input1.mp4 input2.mp4 merged.mp4
		Then if necessary (required if your files had a cover embedded) run:
		> ffmpeg -i merged.mp4 -map 0 -map -0:v -c copy output.mp4	
		
### other useful info
- quick remux can fix file errors
> ffmpeg -i %FILENAME%.ext -c copy %FILENAME%.mp4
> ffprobe -i input.ext -show_entries format=duration -v quiet -of csv=p=0

- this old version of merge_m4b.bat allowed for custom filename entry, edited to only export as input.mp4
> call direnhanced_m4b.bat > fileList.txt
> set /p FILENAME=Type Desired Output file name then hit ENTER to continue...
> ffmpeg -f concat -safe 0 -i fileList.txt -c copy "%FILENAME%.mp4"
> endlocal

- You can double check files with:
> ffmpeg -i file.ext OR ffprobe file.ext

- To export metadata from a file, also have export.bat:
> ffmpeg -i FILE.ext -f ffmetadata in.txt

Alternatively,
> ffmpeg -i FILE.ext -c copy -map_metadata 0 -map_metadata:s:v 0:s:v -map_metadata:s:a 0:s:a -f ffmetadata in.txt

- To remove existing chapters from a file:
> ffmpeg -i input.mp3 -map_metadata -1 -map_chapters -1 -c copy output2.mp3

### Troubleshooting Notes
- This method has trouble with special characters, often in the filename of the chapter files or occasionally in the title fields. If you are having issues, check for these characters first.
- Note that an mp3 id3v2 chapter has a character limit of 62 characters apparently (untested)
- If having performer valdiation issues, make sure cuesheet.rb was replaced (see above)
- If cue is malformed and the audiobook is over 16 hours, make sure cuesheet.rb was replaced; otherwise cue may be improperly formatted
- Tracks in mp3tag must be integers (not fractions or phrases)
- TextToCue.exe is sensitive to format, for example HH:MM:SS must be MM:SS (this should be automatically formatted correctly if using provided export options in mp3tag)
- If Chapter 1 is missing and starts with Chapter 2 then the title field of the file took the title field of first chapter, to fix make sure you have a file TITLE field in cuesheet on first line
- Minor issue, but last chapter duration is often given as 0 if starting from tracklist (I think it is result a result of cue2ffmeta needing a following chapter to calculate end time). Can be fixed with small manual edit, duplicate the final chapter in the tracklist, then delete it in the metadata.txt file before merging with input
- ffmpeg can't handle m4b extension, but changing to mp4 allows it to work
- Had some mp3 files where adding new metadata removed the embedded chapters. This occurred when over 50 chapters and/or 9e7 milliseconds. Apparently was fixed by enforcing id3v2.3 tags, but if issues persist then may have to combine the excess chapters or split into two files.