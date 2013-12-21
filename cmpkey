//Usage: cmpkey keyfile0 keyfile1
//Test whether they are the same

#include <iostream>
#include <cstdio>
#include <algorithm>
#include <cassert>
#include <cmath>

using namespace std;

typedef unsigned char uint8;

#define eps 1e-9
//#define __Debug
#ifdef __Debug
    #define out(x) cout<<(#x)<<"="<<(x)<<endl
#endif
//compare the alphabetic order between A and B, both uint8 of size n
//return 1: A > B, 0: A = B, -1: A < B
inline int cmpr(uint8 *A, uint8 *B, int n){
    for(int i = 0; i < n; i++){
        if(A[i] != B[i]){
            return A[i]>B[i] ? 1 : -1;
        }
    }
    return 0;
}

inline int sgn(float x){
    return fabs(x)<eps ? 0 : (x>0 ? 1 : -1);
}

typedef struct KeypointSt {
    float row, col;             // Subpixel location of keypoint.
    float scale, ori;           // Scale and orientation (range [-PI,PI])
    int block_size;
    uint8 *descrip;     // Vector of descriptor values
    bool operator < (const KeypointSt &rhs) const {
        int t;
        if(block_size != rhs.block_size) return block_size < rhs.block_size;
        else if(t = cmpr(descrip, rhs.descrip, block_size)) return t < 0;
        else if(t = sgn(row - rhs.row)) return t < 0;
        else if(t = sgn(col - rhs.col)) return t < 0;
        else if(t = sgn(scale - rhs.scale)) return t < 0;
        else return sgn(ori - rhs.ori) < 0;
    }
    bool operator != (const KeypointSt &rhs) const {
        #ifdef __Debug
        out(cmpr(descrip, rhs.descrip, block_size));
        out(block_size != rhs.block_size);
        out(sgn(row - rhs.row));
        out(sgn(col - rhs.col));
        out(sgn(scale - rhs.scale));
        out(sgn(ori - rhs.ori));
        if(sgn(ori - rhs.ori))printf("%.6lf %.6lf\n", ori, rhs.ori);
        #endif // __Debug
        return block_size != rhs.block_size
            || sgn(row - rhs.row) || sgn(col - rhs.col)
            || sgn(scale - rhs.scale) || sgn(ori - rhs.ori)
            || cmpr(descrip, rhs.descrip, block_size);
    }
    bool operator == (const KeypointSt &rhs) const {
        return !(*this != rhs);
    }
} *Keypoint;

void ReadKeypointSt(Keypoint p, int block_size, FILE *input){
    fscanf(input, "%f%f%f%f", &p->row, &p->col, &p->scale, &p->ori);
    p->block_size = block_size;
    p->descrip = new uint8[block_size];
    for(int i = 0; i < block_size; i++)
        fscanf(input, "%hhu", p->descrip + i);
}

void FreeKeypointSt(Keypoint p){
    delete [] p->descrip;
}

struct Key {
    int n, block_size;
    KeypointSt *data;
    Key (FILE *input);
    ~Key ();
    bool operator == (const Key &rhs) const;
};

Key::Key(FILE *input){
    fscanf(input, "%d %d", &n, &block_size);
    data = new KeypointSt[n];
    for (int i = 0; i < n; i++)
        ReadKeypointSt(data + i, block_size, input);
    fclose(input);
    sort(data, data + n);
}

Key::~Key(){
    for(int i = 0; i < n; i++)
        FreeKeypointSt(data + i);
    delete [] data;
}

bool Key::operator==(const Key &rhs) const {
    if(n != rhs.n || block_size != rhs.block_size) return false;
    bool f = true;
    for(int i = 0; i < block_size; i++)
        if (data[i] != rhs.data[i]) {
            #ifdef __Debug
            out(i);
            #endif
            i = block_size, f = false;
        }
    return f;
}

//-1: error, 0: not equal, 1: equal
int is_equal(char *f1, char *f2){
    FILE *fp1 = fopen(f1, "r");
    FILE *fp2 = fopen(f2, "r");
    int res = -1;
    if (fp1 == NULL) fprintf(stderr, "Error: cannot access %s\n", f1);
    else if (fp2 == NULL) fprintf(stderr, "Error: cannot access %s\n", f2);
    else {
        Key k1(fp1), k2(fp2);
        res = (k1 == k2) ? 1 : 0;
    }
    if (fp1) fclose(fp1);
    if (fp2) fclose(fp2);
    return res;
}

int main(int argc, char **argv)
{
    if (argc != 3) printf("Usage: cmpkey keyfile0 keyfile1\n");
    else {
        switch(is_equal(argv[1], argv[2])) {
            case 1: printf("%s, %s no difference found\n", argv[1], argv[2]); break;
            case 0: printf("%s, %s is different!\n", argv[1], argv[2]); break;
            default: printf("Something failed!\n"); break;
        }
    }
    return 0;
}
