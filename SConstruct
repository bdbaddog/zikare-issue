#!/usr/bin/env python

import sys
import importlib
import os

# ----------------------------------------------------------------------------------------------- #

def rebase(file_list, old_directory, new_directory):
    """Quick and dirty stand-in to alter the path of a set of files"""

    rebased_file_list = []

    for file in file_list:
        rebased_file_list.append(str(file).replace(old_directory, new_directory))

    return rebased_file_list

# ----------------------------------------------------------------------------------------------- #

environment = Environment()
environment.AppendENVPath('PATH', '/opt/local/bin')

# ----------------------------------------------------------------------------------------------- #
# Step 1: Download the current release

download_url = 'https://github.com/google/googletest/archive/release-1.8.1.tar.gz'

# Determine the target filename for the download (below 'downloads' folder)
archive_filename = os.path.basename(download_url)
archive_file = environment.File(os.path.join('downloads', archive_filename))

# Tell SCons how to "produce" the downloaded archive (by calling wget)
download_archive = environment.Command(
    source = Value(download_url), # The real world script points to a 'download_url' file
    action = 'wget ' + download_url + ' --output-document=$TARGET',
    target = '#/downloads/%s'%archive_filename
)

# ----------------------------------------------------------------------------------------------- #
# Step 2: Extract the release into the build directory

source_files = [
    'build/googletest/src/gtest.cc',
    'build/googletest/src/gtest-death-test.cc',
    'build/googletest/src/gtest-filepath.cc',
    'build/googletest/src/gtest-port.cc',
    'build/googletest/src/gtest-printers.cc',
    'build/googletest/src/gtest-test-part.cc',
    'build/googletest/src/gtest-typed-test.cc'
]

# Tell SCons how to "produce" the sources & headers (by calling tar)
extract_archive = environment.Command(
    source = download_archive,
    action = 'tar --extract --gzip --strip-components=1 --file=$SOURCE --directory=build',
    target = source_files
)

print("Extract_archive: %s"%[e.abspath for e in extract_archive])

# ----------------------------------------------------------------------------------------------- #
# Step 3: Compile the library

if False: # Works: waits for the archive to extract, then compiles
    gtest_environment = environment.Clone()
    gtest_environment.Append(CPPPATH=['build/googletest'])
    gtest_environment.StaticLibrary('bin/gtest', source_files)

else: # Fails: attempts to compile before the archive is extracted
    gtest_environment = environment.Clone()
    gtest_environment.Append(CPPPATH=['build/googletest/include', 'build/googletest'])

    gtest_environment.VariantDir('obj/linux-amd64/src', 'build/googletest/src', duplicate = 0)
    variantdir_source_files = rebase(
        source_files, '#build/googletest/src/', 'obj/linux-amd64/src/'
    )
    print("rebased:%s"%[str(f) for f in variantdir_source_files])
    gtest_environment.StaticLibrary('bin/gtest', variantdir_source_files)
