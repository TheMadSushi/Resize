     /**
     * resize.c
     *
     * Computer Science 50
     * Problem Set 4
     *
     * Resizes a BMP image
     */

    #include <stdio.h>
    #include <stdlib.h>

    #include "bmp.h"

    int main(int argc, char* argv[])
    {

        // ensure proper usage
    if (argc != 4)
        {
            printf("Usage: ./resize n infile outfile\n");
            return 1;
        }

        // remember factor and filenames
        int n = atoi(argv[1]);
        if(n < 0 || n > 100)
        {
            printf("The resize factor should be a positive integer less than 100.\n");
            return 2;
        }
        char* infile = argv[2];
        char* outfile = argv[3];


        // open input file 
        FILE* inptr = fopen(infile, "r");
        if (inptr == NULL)
        {
            printf("Could not open %s.\n", infile);
            return 3;
        }

        // open output file
        FILE* outptr = fopen(outfile, "w");
        if (outptr == NULL)
        {
            fclose(inptr);
            fprintf(stderr, "Could not create %s.\n", outfile);
            return 3;
        }


        // read infile's BITMAPFILEHEADER
        BITMAPFILEHEADER bf;
        fread(&bf, sizeof(BITMAPFILEHEADER), 1, inptr);


        // read infile's BITMAPINFOHEADER
        BITMAPINFOHEADER bi;
        fread(&bi, sizeof(BITMAPINFOHEADER), 1, inptr);


        // ensure infile is (likely) a 24-bit uncompressed BMP 4.0
        if (bf.bfType != 0x4d42 || bf.bfOffBits != 54 || bi.biSize != 40 || 
            bi.biBitCount != 24 || bi.biCompression != 0)
        {
            fclose(outptr);
            fclose(inptr);
            fprintf(stderr, "Unsupported file format.\n");
            return 4;
        }


        //Store old values of width and height before resize
        int width = bi.biWidth;
        int height = bi.biHeight;

        //Resize the width and height
        bi.biWidth = bi.biWidth * n;
        bi.biHeight = bi.biHeight * n;

        // determine padding for scanlines for input file
        int in_padding =  (4 - (width * sizeof(RGBTRIPLE)) % 4) % 4;

        // determine padding for scanlines for output file
        int out_padding =  (4 - (bi.biWidth * sizeof(RGBTRIPLE)) % 4) % 4;


        //Resize the header file info
        bi.biSizeImage = (bi.biWidth * abs(bi.biHeight)) + (out_padding * abs(bi.biHeight));
        bf.bfSize = 54 +  bi.biSizeImage;

        // write outfile's BITMAPFILEHEADER
        fwrite(&bf, sizeof(BITMAPFILEHEADER), 1, outptr);

        // write outfile's BITMAPINFOHEADER
        fwrite(&bi, sizeof(BITMAPINFOHEADER), 1, outptr);



        // iterate over infile's scanlines
        for (int i = 0; i < abs(height); i++)
        {
            //iterate over each scanline n times
            for (int x = 0; x < n; x++)
            {


                // iterate over pixels in scanline
                for (int j = 0; j < width; j++)
                {
                    // temporary storage
                    RGBTRIPLE triple;

                    // read RGB triple from infile
                    fread(&triple, sizeof(RGBTRIPLE), 1, inptr);



                        // write RGB triple to outfile n number of times
                        for(int k = 0; k < n; k++)
                        {
                            fwrite(&triple, sizeof(RGBTRIPLE), 1, outptr);
                        }

                }

                    // skip over padding, if any
                fseek(inptr, in_padding, SEEK_CUR);

                // then add it back (to demonstrate how)
                for (int k = 0; k < out_padding; k++)
                {
                    fputc(0x00, outptr);

                }

                //Go back to the beggining of the input file scanline
                fseek(inptr, -(width * 3 + in_padding ), SEEK_CUR);

            }
            fseek(inptr, width * sizeof(RGBTRIPLE) + in_padding, SEEK_CUR);

        }

        // close infile
        fclose(inptr);

        // close outfile
        fclose(outptr);

        // that's all folks
        return 0;
    }
