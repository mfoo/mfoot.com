---
comments: true
date: 2011-09-04 09:37:45
layout: post
slug: using-compressed-assets-in-librocket-via-physicsfs
title: Using compressed assets in LibRocket via PhysicsFS
wordpress_id: 212
categories:
- Game Development
- Programming
tags:
- assets
- librocket
- physfs
- physicsfs
---

[LibRocket](librocket.com) is a system that allows you to define user interface elements for OpenGL/DirectX applications in HTML and CSS. As well as being easy to get started with and powerful, this is really useful as it means you can load UI elements from files that can be edited without needing to recompile the application. It provides several interface classes that you can customise based on your current platform to allow it to play nice with any other libraries you may be using.

I've been storing compressed project assets using [PhysicsFS](icculus.org/physfs/), which allows for direct read/write access to compressed files such as zip files. After a little fiddling, I produced the class below that allows LibRocket to read it's assets via PhysicsFS.

RocketFileSystemInterface.h:

{% highlight cpp %}
#ifndef _ROCKETFILESYSTEMINTERFACE_H_
#define _ROCKETFILESYSTEMINTERFACE_H_

#include <Rocket/Core/FileInterface.h>

/**
 * A FileInterface for libRocket (http://librocket.com/) that enables the
 * reading of files via PhysicsFS (http://icculus.org/physfs/) directly from a
 * compressed archive.
 *
 * Note: The PhysicsFS system must be initialised before this class is used via
 * PHYSFS_init() and any archives must be made searchable via
 * PHYSFS_addToSearchPath().
 *
 * Author: Martin Foot
 * Date: 4th September 2011
 */
namespace Delta {
class RocketFileSystemInterface : public Rocket::Core::FileInterface {
    public:
        RocketFileSystemInterface();
        ~RocketFileSystemInterface();

        /**
         * Get a read only filehandle to the specified file.
         * @param path  The filename in the compressed archive.
         * @return A Rocket filehandle to the opened file. This can be NULL on
         * failure.
         */
        Rocket::Core::FileHandle Open(const Rocket::Core::String& path);

        /**
         * Close a previously opened file.
         * @param file  The Rocket filehandle to close.
         */
        void Close(Rocket::Core::FileHandle file);

        /**
         * Attempt to read the specified number of bytes into the provided
         * buffer. If the specified number of bytes is greater than the size of
         * the file, only  bytes will be copied.
         * @param buffer    The buffer to read into.
         * @param size  The number of bytes to read.
         * @param file  The Rocket filehandle.
         * @return  The actual number of bytes that was read into the buffer.
         */
        size_t Read(void* buffer, size_t size, Rocket::Core::FileHandle file);

        /**
         * Seek to a point in the previously opened file.
         * @param file  The Rocket filehandle.
         * @param offset    The number of bytes to seek to relative to the
         * supplied origin.
         * @param   One of SEEK_SET (seek relative to the beginning of the file),
         * SEEK_CUR (seek relative to the current position in the file) or
         * SEEK_END (relative to the end of the file).
         * @return  Whether seeking was a success or not.
         */
        bool Seek(Rocket::Core::FileHandle file, long offset, int origin);

        /**
         * Get the length of the file.
         * @param file  The Rocket filehandle.
         * @return  The size in bytes of the file.
         */
        size_t Length(Rocket::Core::FileHandle file);

        /**
         * Get the current read pointer offset.
         * @param file  The Rocket filehandle.
         * @return  The current offset. Returns 0 on failure.
         */
        size_t Tell(Rocket::Core::FileHandle file);
    };
}

#endif // _ROCKETFILESYSTEMINTERFACE_H_
{% endhighlight %}

RocketFileSystemInterface.cpp:

{% highlight cpp %}
#include "RocketFileSystemInterface.h"

#include <Rocket/Core.h>
#include <Rocket/Core/Types.h>
#include <physfs.h>
#include <cstdio>
#include <string>

#include "FileSystemInterface.h"

using Delta::FileSystemInterface;

namespace Delta {

RocketFileSystemInterface::RocketFileSystemInterface() : FileInterface() {};

RocketFileSystemInterface::~RocketFileSystemInterface() {};

Rocket::Core::FileHandle RocketFileSystemInterface::Open(const Rocket::Core::String& path) {
    if (!PHYSFS_exists(path.CString())) {
        return NULL;
    }

    PHYSFS_file* file = PHYSFS_openRead(path.CString());
    return static_cast<PHYSFS_file*>((uintptr_t)file);
}

void RocketFileSystemInterface::Close(Rocket::Core::FileHandle file) {
    PHYSFS_close(static_cast<PHYSFS_file*>((void *)file));
}

size_t RocketFileSystemInterface::Read(void* buffer, size_t size, Rocket::Core::FileHandle file) {
    PHYSFS_file* ptr = static_cast<PHYSFS_file*>((void *) file);

    // Don't read past the end of the file or PhysicsFS will return an error.
    size_t length = PHYSFS_fileLength(ptr);
    if(size > length) {
        size = length;
    }

    size_t read = PHYSFS_read(ptr, buffer, size, 1);

    return read * size;
}

bool RocketFileSystemInterface::Seek(Rocket::Core::FileHandle file, long offset, int origin) {
    PHYSFS_file* ptr = static_cast<PHYSFS_file*>((void *)file);

    int response = 0;

    switch(origin) {
        case SEEK_SET:
            // Seek from the beginning of the file.
            response = PHYSFS_seek(ptr, offset);
            break;
        case SEEK_CUR:
            // Offset from the current position.
            response = PHYSFS_seek(ptr, offset + PHYSFS_tell(ptr));
            break;
        case SEEK_END:
            response = PHYSFS_seek(ptr, PHYSFS_fileLength(ptr));
            break;
    }

    if(response == 0)
        return false;

    return true;
}

size_t RocketFileSystemInterface::Tell(Rocket::Core::FileHandle file) {
    PHYSFS_file* ptr = static_cast<PHYSFS_file*>((void *)file);
    size_t offset = PHYSFS_tell(ptr);
    return offset;
}

size_t RocketFileSystemInterface::Length(Rocket::Core::FileHandle file) {
    PHYSFS_file* ptr = static_cast<PHYSFS_file*>((void *)file);
    return PHYSFS_fileLength(ptr);
}

}
{% endhighlight %}
