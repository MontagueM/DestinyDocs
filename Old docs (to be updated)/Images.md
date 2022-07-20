# Images and Textures

Images in Destiny 2 - whether that's item icons, 3D model textures, or other UI images - are all DDS textures.

DDS (DirectDraw Surface), or DirectXTex texture, is an image container format designed for ease of use with DirectX graphics libraries. The DDS specification only outlines the header layout, with the layout/formatting of the actual texture data determined by the DXGI format in the header. The DXGI format value is an enum that indicates the compression algorith, channel and level layout, and bit-depth of the texture data.

In Destiny 2, a single texture will be split over two different entries:
 - A texture header. This is a stripped down DDS header which contains the width, height, and DXGI format, with some Destiny-specific data such as the file hash for the texture data
 - The actual texture data in whatever format specified by the DXGI format value

To create a valid .dds file, you must:
 1. read the width, height, and DXGI format from the header entry
 2. construct a new DDS header (see resources below)
 3. append the texture data to the DDS header

With the valid .dds file, you should then be able to use external tools to convert the texture to something like a .png file.

Note: the game appears to perform "gamma correction" on the textures, so if you convert as-is the images may appear washed out. I've found success with changing the DXGI format to an sRGB format. e.g:

	headerStream.Seek(0x4, SeekOrigin.Begin);
	var dxgiType = headerBinReader.ReadUInt16();
	var dxgiFormat = (DirectXTexUtility.DXGIFormat)dxgiType;
	
	if (dxgiFormat == DirectXTexUtility.DXGIFormat.R8G8B8A8UNORM) {
		dxgiFormat = DirectXTexUtility.DXGIFormat.R8G8B8A8UNORMSRGB;
	}
	
Resources:
 - [DXGI_FORMAT enumeration](https://docs.microsoft.com/en-au/windows/win32/api/dxgiformat/ne-dxgiformat-dxgi_format?redirectedfrom=MSDN)
 - [DirectXTex texture processing library](https://github.com/Microsoft/DirectXTex/)
 - [A quick class to generate headers, useful when dealing with headerless DDS data or converting game-specific headers to DDS without requiring Microsoft's DirectXTex Library.](https://gist.github.com/Scobalula/d9474f3fcf3d5a2ca596fceb64e16c98)	
