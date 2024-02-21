# allele-matching
make all pairwise matrix and compare them
1-subset the vcf file and select the specific columns
2-remove N from file
3-make the difference and same among the alleles
4-compare the alleles
5-turn into excel and make bar plots 

# 1-select hapmap file and select the columns from 1-5 and 12-75
	cut -f 1-5,12-75 subset_filtered.hmp.txt > new_subset_filtered.hmp.txt

# 2-remove N
	awk -F'\t' 'NR==1 || !/N/' subset_filtered.hmp.txt > output_file.txt

# 3-compare the file from column
	#!/bin/bash
	
	input_file="remove_N.txt"
	
	#Iterate through columns 6 to 69
	for ((i = 6; i <= 69; i++)); do
	    query_column_name=$(awk -F'\t' -v query_column="$i" 'NR == 1 { print $query_column }' "$input_file")
	    output_file="output_${query_column_name}.txt"
	
	    # Run AWK script for each column comparison
	    awk -F'\t' -v compare_column="$i" 'BEGIN {OFS = "\t"} {
	        if (NR == 1) {
	            # Print the header as is
	            print $0;
	        } else {
	            printf "%s", $1; 
	            for (j = 2; j <= 5; j++) {
	                printf "%s%s", OFS, $j;  # Keep columns 1 to 5 the same
	            }
	            for (j = 6; j <= 69; j++) {
	                if (j == compare_column) {
	                    printf "%s%s", OFS, $j;  # Keep the value of compare_column unchanged
	                } else {
	                    printf "%s%s", OFS, ($compare_column == $j) ? "same" : "different";
	                }
	            }
	            print "";
	        }
	    }' "$input_file" > "$output_file"
	done

# 4-count the different and same in a individual file
	awk -F'\t' '{
	    if (NR == 1) {
	        # Print the header
	        printf "column header\tdifferent\tsame\n";
	        for (i = 1; i <= NF; i++) {
	            header[i] = $i;
	            counts[i]["same"] = 0;
	            counts[i]["different"] = 0;
	        }
	    } else {
	        # Iterate through each column
	        for (i = 1; i <= NF; i++) {
	            counts[i][$i]++;
	        }
	    }
	} END {
	    # Print the counts for each column
	    for (i = 1; i <= NF; i++) {
	        printf "%s\t%d\t%d\n", header[i], counts[i]["different"], counts[i]["same"];
	    }
	}' output_Zamorano_fa3653d3-96e6-4b45-bd60-03921a96a590_[Guanhuabai].txt > zamoranioutput_counts.txt


# 4-count the number of difference and same in a loop for multiple files
	#!/bin/bash
	
	input_dir="/ibex/project/c2141/dragon-fruit/genotyped/hapmap/hapmap/tassel/all_comparison/remove_N/"
	output_dir="/ibex/project/c2141/dragon-fruit/genotyped/hapmap/hapmap/tassel/all_comparison/compared/"
	
	#Create the output directory if it doesn't exist
	mkdir -p "${output_dir}"
	
	#Iterate over files in the input directory
	for input_file in "${input_dir}"/*.txt; do
	    # Extract the file name without extension
	    file_name=$(basename "${input_file}" .txt)
	
	    # Define the output file path
	    output_file="${output_dir}/${file_name}_compared.txt"
	
	    # Apply AWK script to the current input file and save results to the output file
	    awk -F'\t' '{
	        if (NR == 1) {
	            # Print the header
	            printf "column header\tdifferent\tsame\n";
	            for (i = 1; i <= NF; i++) {
	                header[i] = $i;
	                counts[i]["same"] = 0;
	                counts[i]["different"] = 0;
	            }
	        } else {
	            # Iterate through each column
	            for (i = 1; i <= NF; i++) {
	                counts[i][$i]++;
	            }
	        }
	    } END {
	        # Print the counts for each column
	        for (i = 1; i <= NF; i++) {
	            printf "%s\t%d\t%d\n", header[i], counts[i]["different"], counts[i]["same"];
	        }
	    }' "${input_file}" > "${output_file}"
	
	    echo "Processed: ${input_file} -> ${output_file}"
	done


 
