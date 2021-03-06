# **九大排序算法**

1. **交换排序（冒泡排序）**

2. **插入排序**

3. **选择排序**

4. **快速排序（分治思想）**
* 思想：1、在无序数组中选中一个数作为轴，2、将小于这个轴的数放在轴的左侧区域，大于这个数的放在右侧区域 3、对左右区域重复上述两个步骤，直到数组有序
* 代码
```
class Solution {
public:
    vector<int> sortArray(vector<int>& nums) {
        if (nums.size()==0)     
            return nums;
        quickSort(nums,0,nums.size()-1);
        return nums;
    }

    void quickSort(vector<int>& nums,int start,int end){
        if (start>=end)
            return;
        int mid = partition(nums,start,end);
        quickSort(nums,start,mid-1);
        quickSort(nums,mid+1,end);
    }

    int partition(vector<int>& nums,int start,int end){
        int randomIndex = rand()%(end-start+1)+start;       //随机选择区间内的一个数作为基数，可以防止数组有序情况下时间复杂度退化到O(n*n)
        swap(nums[start],nums[randomIndex]);
        int pivot = nums[start];
        int left = start+1, right = end;
        while(left<right){
            while(left<right && nums[left]<pivot)   left++;
            while(left<right && nums[right]>pivot)  right--;
            if (left<right){
                swap(nums[left],nums[right]);
                left ++;
                right--;
            }
        }
        if (left==right && nums[right]>pivot)   
            right--;
        swap(nums[start],nums[right]);
        return right;
    }

};
```
5. **归并排序(分治思想)**
* 思想：利用二分思想，将将无序数列一分为二，分别对两个数列进行归并排序，然后合并两个有序数列为一个有序数列
* 数组中的归并排序
```
//采用了原地数组合并的方法
class Solution {
public:
    vector<int> sortArray(vector<int>& nums) {
        mergeSort(nums,0,nums.size()-1);
        return nums;
    }

    void mergeSort(vector<int>& nums,int start,int end){
        if (start==end)
            return;
        int mid = (start+end)/2;
        mergeSort(nums,start,mid);
        mergeSort(nums,mid+1,end);
        merge(nums,start,mid,end);
    }

    void merge(vector<int>& nums,int start,int mid,int end){
        int start1 = start,end1 = mid;
        int start2 = mid+1,end2 = end;
        while (start1 <= end1 && start2 <= end2){
            if (nums[start2]>=nums[start1])
                start1++;
            else{
                int temp = nums[start2];
                for (int i=start2;i>start1;i--)
                    nums[i] = nums[i-1];
                nums[start1] = temp;
                start1 ++;
                start2 ++;
                end1 ++;               
            }
        }
    }
};
```
```
//借用了一个额外的数组来合并两个数组
class Solution {
public:
    vector<int> sortArray(vector<int>& nums) {
        vector<int> result(nums.size(),0);
        mergeSort(nums,0,nums.size()-1,result);
        return nums;
    }

    void mergeSort(vector<int>& nums,int start,int end,vector<int>& result){
        if (start==end)
            return ;
        int mid = (start+end)/2;
        mergeSort(nums,start,mid,result);
        mergeSort(nums,mid+1,end,result);
        merge(nums,start,mid,end,result);
    }

    void merge(vector<int>& nums,int start,int mid,int end,vector<int>& result){
        int start1 = start,end1 = mid;
        int start2 = mid+1,end2 = end;
        int resultIndex = start1;
        while(start1<=end1 && start2<=end2){
            if (nums[start1]<nums[start2])
                result[resultIndex++] = nums[start1++];
            else
                result[resultIndex++] = nums[start2++];
        }
        while(start1<=end1)
            result[resultIndex++] = nums[start1++];
        while(start2<=end2)
            result[resultIndex++] = nums[start2++];
        
        for(int i=start;i<=end;i++)
            nums[i] = result[i];
    }
};
```
* 链表中的归并排序
```
class Solution {
public:
    ListNode* sortList(ListNode* head) {
        if (!head || !head->next)   return head;
        ListNode * mid=endOfhalf(head);
        ListNode* startOfSecond=mid->next;

        mid->next=nullptr;

        ListNode* p1=sortList(head);
        ListNode* p2=sortList(startOfSecond);
        return mergeTwoLists(p1,p2);
    }

    ListNode* endOfhalf(ListNode* head){
        if (head==nullptr)  return nullptr;

        ListNode* fast=head,*slow=head;
        while (fast->next!=nullptr && fast->next->next!=nullptr){
            fast=fast->next->next;
            slow=slow->next;
        }
        return slow;
    }
    
    ListNode* mergeTwoLists(ListNode* l1, ListNode* l2) {
        ListNode* dummy=new ListNode(0,nullptr);
        ListNode* p=dummy;
        while(l1 && l2){
            if (l1->val<l2->val){
                p->next=l1;
                p=l1;
                l1=l1->next;
            }
            else{
                p->next=l2;
                p=l2;
                l2=l2->next;
            }
        }
        if(l1!=nullptr) p->next=l1;
        if(l2!=nullptr) p->next=l2;
        return dummy->next;
    }
};
```
6. **堆排序**
```
class Solution {
public:
    vector<int> sortArray(vector<int>& nums) {
        buildMaxHeap(nums);
        for (int i=nums.size()-1;i>0;i--){
            swap(nums[0],nums[i]);
            maxHeapAdjust(nums,0,i);
        }
        return nums;
    }

    void buildMaxHeap(vector<int>& nums){
        for (int i = nums.size()/2-1;i>=0;i--)
            maxHeapAdjust(nums,i,nums.size());
    }

    void maxHeapAdjust(vector<int>& nums,int i,int heapsize){
        int left = 2*i+1,right = left+1;
        int max_index = i ;
        if (left<heapsize)
            max_index = nums[left]>nums[max_index]?left:max_index;
        if (right<heapsize)
            max_index = nums[right]>nums[max_index]?right:max_index;
        if (max_index!=i){
            swap(nums[i],nums[max_index]);
            maxHeapAdjust(nums,max_index,heapsize);
        }
    }
};
```
7. **希尔排序**
* 思想：shell排序的本质是多间隔的插入排序，以一个递减的增量序列来进行插入排序，最后一个增量为1就是一个普通的插入排序。
* 平均时间复杂度n的1.3次方，突破n的2次方在于一次交换可以消除多对逆序对，而排序的本质就是消除逆序对的过程，普通的排序算法，一次交换最多消除一个逆序对
8. **基数排序**

9. **桶排序**
