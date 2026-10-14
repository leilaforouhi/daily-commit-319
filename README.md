
import hashlib
from pathlib import Path


def calculate_hash(file_path):
    digest = hashlib.sha256()

    with open(file_path, "rb") as file:
        for chunk in iter(lambda: file.read(8192), b""):
            digest.update(chunk)

    return digest.hexdigest()


def find_duplicate_files(directory):
    files_by_hash = {}

    for path in Path(directory).rglob("*"):
        if not path.is_file():
            continue

        try:
            file_hash = calculate_hash(path)
            files_by_hash.setdefault(file_hash, []).append(path)
        except OSError as error:
            print(f"Could not read {path}: {error}")

    return [
        paths
        for paths in files_by_hash.values()
        if len(paths) > 1
    ]


if __name__ == "__main__":
    folder = input("Enter the folder path to scan: ").strip()
    directory = Path(folder).expanduser()

    if not directory.is_dir():
        print("Error: Please enter a valid directory.")
    else:
        duplicates = find_duplicate_files(directory)

        if not duplicates:
            print("No duplicate files found.")
        else:
            for group_number, paths in enumerate(duplicates, start=1):
                print(f"\nDuplicate group {group_number}:")
                for path in paths:
                    print(f"  {path}")
