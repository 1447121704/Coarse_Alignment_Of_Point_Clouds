# 3D Coarse Alignment of Point Clouds For Pose Estimation & Workpiece Localization

## Motivation

3D Vision Sensors can generate depth map, using the difference in location of an observed object (disparity) between the left and right camera views to measure the depth of the object from the viewer. This adds an additional dimensionality to a 2d image making it as a 3d image. This feature could be used to find the distance between objects, separating objects from the background layers behind them and much better object recognition than a traditional camera – well suits for computer vision systems in robots. 

Welding is an essential process in building machines and industrial products. Typically done by small and medium sized companies(SME), it requires skilled labor and experience. Aim of welding robots is to make this process cost effective and less strenuous. As programming of welding robots in online mode is a time consuming process, necessity of offline programming arises. This topic is an approach to localize the workpiece thereby achieving fully offline programming of welding tasks. Though this process is not specific to any application, the motivation behind is to localize objects by a 3D vision system.

## Goal of the Topic

- Coarse alignment of CAD point cloud and Camera Measurement (Registration of Point Cloud).
- Estimate Rigid transformation between Camera Coordinates and Workpiece coordinates.
- With initial pose estimation , achieve accurate workpiece localization.

## PCL Library

The Point Cloud Library (PCL) is a standalone, large scale, open project for 2D/3D image and point cloud processing. PCL has various methods processing point clouds and has the support of visualization in VTK. This work includes PCL (C++) version 1.8 integrated with Visual Studio 2017. More information is available on the site http://www.pointclouds.org/documentation/tutorials/

## Steps Involved

- Dataset Reduction
- Feature Generation
- Model Training & Evaluation
- Results Obtained

#### Dataset Reduction - Keypoint Generation

A point cloud is a collection of data points defined by a given coordinates system. In a 3D coordinates system, for example, a point cloud may define the shape of some real or created physical system. Usually point clouds are massive in count i.e based on the camera parameters and density of measurement setting, point clouds would be huge in numbers. Basic approach used in this topic is to find corresponding points in both the point clouds and thereby estimating rigid transformation between corresponding points. But having a huge dataset makes computation much difficult. So PCL Keypoint generation is applied to both the point clouds to minimize the region of interest. There are various keypoint methods discussed in PCL library such as SIFT, NARF, ISS. For this problem statement, I have found that Harris 3d keypoint generation gave better results.

The Harris method (Harris and Stephens, 1988) is a corner and edge based method and these types of methods are characterized by their high-intensity changes in the horizontal and vertical directions. For the 3D case, the adjustment made in PCL for the Harris3D detector replaces the image gradients by surface normals. With that, they calculate the covariance matrix around each point.

pcl::HarrisKeypoint3D<pcl::PointXYZ, pcl::PointXYZI>* harris3D = new
			pcl::HarrisKeypoint3D<pcl::PointXYZ, pcl::PointXYZI>(pcl::HarrisKeypoint3D<pcl::PointXYZ, pcl::PointXYZI>::HARRIS);

		//Code Snippet to Detect Harris 3D Keypoints
    
    harris3D->setNonMaxSupression(false);
		harris3D->setRadius(12);
		harris3D->setInputCloud(orig_pcd);
		pcl::PointCloud<pcl::PointXYZI>::Ptr keypoints(new pcl::PointCloud<pcl::PointXYZI>);
		pcl::PointCloud<pcl::PointXYZ>::Ptr key_regions(new pcl::PointCloud<pcl::PointXYZ>);
		harris3D->compute(*keypoints);
		pcl::StopWatch watch;
		pcl::console::print_highlight("Detected %zd points in %lfs\n", keypoints->size(), watch.getTimeSeconds());

		/*if (!keypoints->size())
		{*/
		pcl::io::savePCDFile("keypoints_10r_cad_pose0.pcd", *keypoints);
		pcl::console::print_info("Saved keypoints to keypoints.pcd\n");
		int size_pt = 0;
		float intensity_thresh = .0116f;
		for (size_t i = 0; i < keypoints->size(); i++)
		{
			if (keypoints->points[i].intensity >= intensity_thresh)
			{
				++size_pt;
			}
		}
		key_regions->width = size_pt;
		key_regions->height = 1;
		key_regions->is_dense = false;
		key_regions->points.resize(key_regions->width * key_regions->height);
		int kex_index = 0;
		for (size_t i = 0; i < keypoints->size(); i++)
		{
			
			if (keypoints->points[i].intensity >= intensity_thresh)
			{
				key_regions->points[kex_index].x = keypoints->points[i].x;
				key_regions->points[kex_index].y = keypoints->points[i].y;
				key_regions->points[kex_index].z = keypoints->points[i].z;
				++kex_index;
			}
		}
		pcl::io::savePCDFile("key_regions.pcd", *key_regions);
    
    
