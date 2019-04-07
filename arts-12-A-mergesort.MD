# ***�鲢�����Javaʵ��
�鲢����ʹ���˷��ε�˼�룬���´���ؼ�Ϊ��

1,partition�еݹ�Ķ����ݽ��з�������:���������Ϊ������ֻ�����������˾ͱ�����кϲ�������

2,merge�����ж����ݽ�������

����Ϊ����ʵ�֣�
```
package algorithm;

/**
 * �鲢����ʵ��
 * @author xuqk
 *
 */
public class MergeSort {
    public void sort(int[] data){
        partition(data,0,data.length -1);
    }
    
    public void partition(int[] data,int l, int r){
        //�ݹ����ֹ�����������ص㡣������ֻ������Ԫ��ʱ���ؽ��кϲ�
        if(l >= r){
            return;
        }
        
        int q = (l + r) /2;
        partition(data,l,q);
        partition(data,q+1,r);
        
        merge(data,l,q,r);//�ϲ�
    }

    /**
     * �ϲ�
     * @param data
     * @param l �������±�
     * @param q ÿ�μ�����м�ֵ
     * @param r �������±�
     */
    private void merge(int[] data,int l, int q,int r) {
        int[] mergedArray = new int[(r - l) + 1];
        int i =l; //�����������ı������±�
        int j = q+1; //�����������ı������±�
        int k =0;
        
        //�鲢�����е���������merge�Ĺؼ�����
        while(i <=q & j <= r){
            if(data[i] <= data[j] ){
                mergedArray[k++] = data[i++];
            }else{
                mergedArray[k++] = data[j++];
            }
        }
        
        //���������ϲ�ʣ�������
        //Ĭ����Ϊ��������������
        int start = i;
        int end = q;
        if(j <= r){ //�ұ������ ��������
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
     * ������
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