# audioPCMtoMP3  
iOS 录音后的音频文件，转码，转成mp3格式，pcm->mp3, wav->mp3,使用以下方法
### usage:
```
#import "lame.h"
-(NSString *)audioPCMtoMP3:(NSString *)wavPath toFile:(NSString *)mp3Path
{
    // wavPath 被转换的音频文件位置，
    // 比如这样：/var/mobile/Containers/Data/Application/AA6D7D77-089E-4F93-84CF-4E944EA94298/Library/Caches/a.wav
    // mp3Path 输出生成的Mp3文件位置，
    // 比如这样：/var/mobile/Containers/Data/Application/AA6D7D77-089E-4F93-84CF-4E944EA94298/Library/Caches/b.mp3
    // 另外要注意，码率的设置
    /*
    NSArray *paths1 = NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES);
    [paths1 objectAtIndex:0]
    NSArray *paths2 = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES);
    [paths2 objectAtIndex:0]
    */
    @try {
        int read, write;
        
        FILE *pcm = fopen(wavPath.UTF8String, "rb");
        fseek(pcm, 4*1024, SEEK_CUR); //skip file header
        FILE *mp3 = fopen(mp3Path.UTF8String, "wb");
        
        const int PCM_SIZE = 8192;
        const int MP3_SIZE = 8192;
        short int pcm_buffer[PCM_SIZE*2];
        unsigned char mp3_buffer[MP3_SIZE];
        
        lame_t lame = lame_init();
        lame_set_in_samplerate(lame, 44100.0); //注意这里的码率设置，要与wavPath文件的码率保持一致
        lame_set_VBR(lame, vbr_default);
        lame_init_params(lame);
        
        do {
            read = (int)fread(pcm_buffer, 2*sizeof(short int), PCM_SIZE, pcm);
            if (read == 0)
                write = lame_encode_flush(lame, mp3_buffer, MP3_SIZE);
            else
                write = lame_encode_buffer_interleaved(lame, pcm_buffer, read, mp3_buffer, MP3_SIZE);
            
            fwrite(mp3_buffer, write, 1, mp3);
            
        } while (read != 0);
        
        lame_close(lame);
        fclose(mp3);
        fclose(pcm);
    }
    @catch (NSException *exception) {
        NSLog(@"%@",[exception description]);
    }
    @finally {
        NSLog(@"MP3生成成功");
    }
}
```
