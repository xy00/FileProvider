# FileProvider (experimental)

This Swift library provide a swifty way to deal with local and remote files and directories in same way. This library provides implementaion of WebDav and SMB/CIFS (incomplete) and local files.

All functions are async calls and it wont block your main thread.

## Installation

#### Manually
Copy Source folder to your project!

#### Git clone
Use this command on terminal to get a clone:

	git clone https://github.com/amosavian/FileProvider FileProvider

#### Submodule into your project
Use this command if you have a git based project in your projects directory:

	git submodule add https://github.com/amosavian/FileProvider FileProvider

#### Cocoapods / Carthage / Swift Package Manager

I will add when project is completed is ready to use in production envioronment

## Usage

Each provider has a specific class which conforms to FileProvider protocol and share same syntax

### Providers

For now this providers are supported:

**LocalFileProvider :** a wrapper for `NSFileManager` with some additions like searching and reading a portion of file

**WebDAVFileProvider :** WebDAV protocol is usual file transmission system on Macs

**SMBFileProvider :** SMB/CIFS and SMB2/3 are file and printer sharing protocol which is originated from Windows and SMB2/3 is now replacing AFP protocol on MacOS. I Implemented data types and some basic functions but *main interface is not implemented yet!*

**FTPFileProvider :** not implemented yet!

**DropboxFileProvider :** not implemented yet!

Your pull requests are welcomed!

### Initialization

For LocalFileProvider if you want to deal with `Documents` folder

	let documentsFileProvider = LocalFileProvider()

is equal to:
	    
	let documentPath = NSSearchPathForDirectoriesInDomains(NSSearchPathDirectory.DocumentDirectory, NSSearchPathDomainMask.UserDomainMask, true);
	let documentsURL = NSURL(fileURLWithPath: documentPath);
	let documentsFileProvider = LocalFileProvider(baseURL: documentsURL)

You can't change the base url later. and all paths are related to this base url by default.

For remote file providers authentication may be necessary:

	let credential = NSURLCredential(user: "user", password: "pass", persistence: NSURLCredentialPersistence.Permanent)
	let webdavProvider = WebDAVFileProvider(baseURL: "http://www.example.com/dav", credential: credential)
	
For interaction with UI, set delegate variable of `FileProvider` object

### Delegate

For updating User interface please consider using delegate method instead of completion handlers. Delegate methods are guaranteed to run in main thread to avoid bugs.

It's simply tree method which indicated whether the operation failed, succeed and how much of operation has been done (suitable for uploading and downloading operations).

Your class should conforms `FileProviderDelegate` class:

	override func viewDidLoad() {
		documentsFileProvider.delegate = self
	}
	
	func fileproviderSucceed(fileProvider: FileProvider, operation: FileOperation) {
		switch operation {
		case .Copy(source: let source, destination: let dest):
			NSLog("\(source) copied to \(dest).")
		case .Remove(path: let path):
			NSLog("\(path) has been deleted.")
		default:
			break
		}
	}
	
    func fileproviderFailed(fileProvider: FileProvider, operation: FileOperation) {
    	switch operation {
		case .Copy(source: let source, destination: let dest):
			NSLog("copy of \(source) failed.")
		case .Remove(path: let path):
			NSLog("\(path) can't be deleted.")
		default:
			break
		}
    }
	
    func fileproviderProgress(fileProvider: FileProvider, operation: FileOperation, progress: Float) {
		switch operation {
		case .Copy(source: let source, destination: let dest):
			NSLog("Copy\(source) to \(dest): \(progress * 100) completed.")
		default:
			break
		}
	}


Use completion handlers for error handling or result processing as far as possible.

### Directory contents and file attributes

There is a `FileObject` class which holds file attributes like size and creation date. You can retrieve information of files inside a directory or get information of a file directly

	documentsFileProvider.attributesOfItemAtPath(path: "/file.txt", completionHandler: {
	    (attributes: LocalFileObject?, error: ErrorType?) -> Void} in
		if let attributes = attributes {
			print("File Size: \(attributes.size)")
			print("Creation Date: \(attributes.createdDate)")
			print("Modification Date: \(modifiedDate)")
			print("Is Read Only: \(isReadOnly)")
		}
	)

	documentsFileProvider.contentsOfDirectoryAtPath(path: "/", 	completionHandler: {
	    (contents: [LocalFileObject], error: ErrorType?) -> Void} in
		for file in contents {
			print("Name: \(attributes.name)")
			print("Size: \(attributes.size)")
			print("Creation Date: \(attributes.createdDate)")
			print("Modification Date: \(modifiedDate)")
		}
	)

### Change current directory

	documentsFileProvider.currentPath = "/New Folder"
	// now path is ~/Documents/New Folder

### Creating File and Folders

Creating new directory:

	documentsFileProvider.createFolder(folderName: "new folder", atPath: "/", completionHandler: nil)

Creating new file from data stream:

	let data = "hello world!".dataUsingEncoding(NSUTF8StringEncoding)
	let file = FileObject(name: "old.txt", createdDate: NSDate(), modifiedDate: NSDate(), isHidden: false, isReadOnly: true)
	documentsFileProvider.createFile(fileAttribs: file, atPath: "/", contents: data, completionHandler: nil)

### Copy and Move/Rename Files

	// Copy file old.txt to new.txt in current path
	documentsFileProvider.copyItemAtPath(path: "new folder/old.txt", toPath: "new.txt", overwrite: false, completionHandler: nil)

	// Move file old.txt to new.txt in current path
	documentsFileProvider.moveItemAtPath(path: "new folder/old.txt", toPath: "new.txt", overwrite: false, completionHandler: nil)

### Delete Files

	documentsFileProvider.removeItemAtPath(path: "new.txt", completionHandler: nil)

***Caution:*** This method will not delete directories with content.


### Retrieve Content of File

THere is two method for this purpose, one of them loads entire file into NSData and another can load a portion of file.

	documentsFileProvider.contentsAtPath(path: "old.txt:, completionHandler: {
		(contents: NSData?, error: ErrorType?) -> Void
		if let contents = contents {
			print(String(data: contents, encoding: NSUTF8StringEncoding)) // "hello world!"
		}
	})
	
If you want to retrieve a portion of file you should can `contentsAtPath` method with offset and length arguments. Please note first byte of file has offset: 0.

	documentsFileProvider.contentsAtPath(path: "old.txt", offset: 2, length: 5, completionHandler: {
		(contents: NSData?, error: ErrorType?) -> Void
		if let contents = contents {
			print(String(data: contents, encoding: NSUTF8StringEncoding)) // "llo w"
		}
	})

### Write Data To Files

	let data = "What's up Newyork!".dataUsingEncoding(NSUTF8StringEncoding)
	documentsFileProvider.writeContentsAtPath(path: "old.txt", contents data: data, atomically: true, completionHandler: nil)


### TODO List

- [ ] TCPSocket test (using telnet)
- [ ] SMB2 protocol support using TCPSocketClient
- [ ] FTP/FTPS protocol support using TCPSocketClient
- [ ] Dropbox support
- [ ] Amazon S3 support
- [ ] SMB1 protocol implementation