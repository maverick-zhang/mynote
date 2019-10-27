```python
def get_serializer_class(self): 
    if self.action == "retrieve":      
        return UserDetailSerializer    
    elif self.action == "create":        
        return UserRegSerializer   
    return UserDetailSerializer
```

