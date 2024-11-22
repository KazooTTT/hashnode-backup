---
title: "在vtk.js中stl和json的互相转化"
datePublished: Fri Nov 22 2024 06:41:00 GMT+0000 (Coordinated Universal Time)
cuid: cm3sdhh7r000008jx079vbnjm
slug: vtkjsstljson

---

## stl如何转为json

```ts
import vtkSTLReader from '@kitware/vtk.js/IO/Geometry/STLReader';

const getStlModelFromPath = async (path: string) => {
  const response = await fetch(path);
  const stlArrayBuffer = await response.arrayBuffer();
  
  const stlReader = vtkSTLReader.newInstance();
  stlReader.parseAsArrayBuffer(stlArrayBuffer);
  
  const polyData = stlReader.getOutputData();
  return polyData;
};

const stlPath = '/path/to/your/model.stl';
const polyData = await getStlModelFromPath(stlPath);
const jsonData = polyData.toJSON();
```

## json如何转为stl

```ts
import modelJSON from './model.json';

const convertPolyDataJSONToStl = (polyDataJSON: string, fileName: string = 'model.stl') => {
    const polyData = vtkPolyData.newInstance(polyDataJSON);
    const writer = vtkSTLWriter.newInstance();

    writer.setInputData(polyData);
    const fileContents = writer.getOutputData();

    // Create a blob and download link
    const blob = new Blob([fileContents], { type: 'application/octet-stream' });
    const a = window.document.createElement('a');
    a.href = window.URL.createObjectURL(blob);
    a.download = fileName;

    // Trigger download
    document.body.appendChild(a);
    a.click();
    document.body.removeChild(a);
    window.URL.revokeObjectURL(a.href);
};


convertPolyDataJSONToStl(modelJSON);
```