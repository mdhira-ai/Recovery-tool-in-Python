## *creating a basic recovery tool in Python:*

 -  install Necessary Libraries:
> pip install hachoir construct

    import hachoir
    import construct
    import os
    
    disk_path = "/dev/sda1"  # Replace with the correct path
    
    # Scan for Deleted Files
    
    def scan_for_deleted_files(disk_path):
        try:
            # Determine file system type
            file_system = hachoir.parser.guessParser(disk_path)
    
            # Parse file system metadata to identify deleted files
            deleted_files_info = file_system.parse_deleted_files()
    
            return deleted_files_info
    
        except hachoir.parser.Error as e:
            print("Error parsing disk:", e)
            return None
    
    # Recover a Specific File
    
    def recover_file(disk_path, file_info):
        # Create parser for the file's format
        file_parser = hachoir.parser.createParser(file_info.file_format)
    
        # Open the file on disk
        with open(disk_path, "rb") as disk_file:
            disk_file.seek(file_info.offset)  # Seek to file's starting position
    
            try:
                # Parse file blocks
                file_parser.parse(disk_file)
    
                # Extract file data
                file_data = file_parser.extract()
    
                # Save recovered file
                with open(os.path.join("recovered_files", file_info.filename), "wb") as recovered_file:
                    recovered_file.write(file_data)
    
                print("Recovered file:", file_info.filename)
    
            except construct.core.ConstError as e:
                print("Error parsing file:", e)
    
    # Main Program
    
    if __name__ == "__main__":
        deleted_files_info = scan_for_deleted_files(disk_path)
    
        if deleted_files_info:
            os.makedirs("recovered_files", exist_ok=True)  # Create output directory
    
            for file_info in deleted_files_info:
                recover_file(disk_path, file_info)
    
        else:
            print("No deleted files found.")



    


