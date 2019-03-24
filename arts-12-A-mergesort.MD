# ***归并排序的Java实现
归并排序使用了分治的思想，如下代码关键为：

1,partition中递归的对数据进行分区处理:其结束条件为数组中只有两个数组了就必须进行合并排序了

2,merge方法中对数据进行排序。

如下为代码实现：
```
package algorithm;

/**
 * 归并排序实现
 * @author xuqk
 *
 */
public class MergeSort {
    public void sort(int[] data){
        partition(data,0,data.length -1);
    }
    
    public void partition(int[] data,int l, int r){
        //递归的终止条件，这是重点。在数据只有两个元素时返回进行合并
        if(l >= r){
            return;
        }
        
        int q = (l + r) /2;
        partition(data,l,q);
        partition(data,q+1,r);
        
        merge(data,l,q,r);//合并
    }

    /**
     * 合并
     * @param data
     * @param l 数组左下标
     * @param q 每次计算的中间值
     * @param r 数组右下标
     */
    private void merge(int[] data,int l, int q,int r) {
        int[] mergedArray = new int[(r - l) + 1];
        int i =l; //处理左边数组的变量的下标
        int j = q+1; //处理左边数组的变量的下标
        int k =0;
        
        //归并过程中的排序：这是merge的关键操作
        while(i <=q & j <= r){
            if(data[i] <= data[j] ){
                mergedArray[k++] = data[i++];
            }else{
                mergedArray[k++] = data[j++];
            }
        }
        
        //处理上述合并剩余的数据
        //默认认为左边数组大还有数据
        int start = i;
        int end = q;
        if(j <= r){ //右边数组大 还有数据
            start = j;
            end = r;
        }
        while(start <= end){
            mergedArray[k++] = data[start++];
        }
        
        for(int index=0;index <= (r - l);index++){
            data[l + index] = mergedArray[index];
        }
        
        return;
    }
    
    /**
     * 测试用
     * @param args
     */
    public static void main(String[] args) {
        MergeSort mergeSort = new MergeSort();
        int testArray[] = new int[]{199,4,78,23,58,999,1024,6,8,2,201};
        for(int i=0;i < testArray.length;i++ ){
            System.out.print(testArray[i]);
            if(i == (testArray.length -1)){
                System.out.println("");
            }else{
                System.out.print("->");
            }
        }
        mergeSort.sort(testArray);

        System.out.println("-----------------------");
        for(int i=0;i < testArray.length;i++ ){
            System.out.print(testArray[i]);
            System.out.print("->");
        }
    }
}

```
