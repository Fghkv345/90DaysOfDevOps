# 1. Create the file
touch notes.txt

# 2. Write the first line (overwrites/creates the file)
echo "Line 1: Linux file practice" > notes.txt

# 3. Append the second line
echo "Line 2: Using redirection" >> notes.txt

# 4. Append the third line and display it at the same time
echo "Line 3: Reading files with cat" | tee -a notes.txt

# 5. Read the complete file
cat notes.txt

# 6. Read the first 2 lines
head -n 2 notes.txt

# 7. Read the last 2 lines
tail -n 2 notes.txt
