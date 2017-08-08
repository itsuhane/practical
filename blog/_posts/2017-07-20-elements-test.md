---
title: "Element Tests"
---

Below are tests of all elements including some common combinations.

# h1 title

h1 title is the biggest title. It is used at page titles, post titles, etc.
h1 titles are 2-times standard size, and have 0.5 standard line-height margin for top and bottom.
The only exception is page title/post title, they have no bottom margins.

## h2 title

h2 title is the second biggest title.
It can be used as the first-rank title in the body of an article.
h2 titles have 0.5 standard line-height margin for top and bottom.

### h3 title

h3 titles can be used as the secondary title in article.
h3 titles have 0.5 standard line-height top margin.

#### h4 title

h4 title is even smaller than h3 title.
h4 titles have 0.5 standard line-height top margin.

##### h5 title

h5 titles have the same size as normal text. It is in strong style and have 0.5 standard line-height top margin.

###### h6 title

h6 titles are smaller than normal text.

---

The line above and below are two separators.

---

[Links](#) are very useful. However, they have different appearance in different locations.
In article, links are underlined to give a little hint. In other places, they have no underline.
Be aware that even for titles, if it is in article content, links in it will be underlined.

### For example, this is an h3 title with [link](#) in it.

Below is a piece of code

``` cpp
template <typename Mesh>
class CurvatureEstimator {
public:
  typedef typename Mesh::Scalar Scalar;
  typedef typename Mesh::Normal Normal;

  typedef typename Mesh::VertexHandle VertexHandle;
  typedef typename Mesh::VertexIter VertexIter;
  typedef typename Mesh::VertexOHalfedgeCCWIter VertexOHalfedgeCCWIter;

  typedef typename Mesh::HalfedgeHandle HalfedgeHandle;

  static void run(Mesh &mesh) {
    mesh.update_normals();
    for (VertexIter vi = mesh.vertices_sbegin(); vi != mesh.vertices_end(); ++vi) {
      estimate_vertex_curvature(mesh, *vi);
    }
  }

  static void estimate_vertex_curvature(Mesh &mesh, const VertexHandle &vh) {
    typedef Eigen::Matrix<Scalar, 3, 3> Matrix3S;
    Matrix3S tensor3;
    tensor3.setZero();
    for (VertexOHalfedgeCCWIter vohei = mesh.voh_ccwbegin(vh); vohei != mesh.voh_ccwend(vh); ++vohei) {
      HalfedgeHandle ohe = *vohei;
      HalfedgeHandle ihe = mesh.opposite_halfedge_handle(ohe);

      if (mesh.is_boundary(ohe) || mesh.is_boundary(ihe)) continue;

      Normal edge_vector = mesh.calc_edge_vector(ohe);
      Scalar edge_length = edge_vector.norm();
      edge_vector.normalize_cond(); // edge_vector is normalized!

      Normal normal1 = mesh.normal(mesh.face_handle(ohe));
      Normal normal2 = mesh.normal(mesh.face_handle(ihe));

      Scalar sinus = OpenMesh::dot(OpenMesh::cross(normal1, normal2), edge_vector);
      Scalar beta = asin(OpenMesh::sane_aarg(sinus));

      Eigen::Matrix<Scalar, 3, 1> eev{ edge_vector[0], edge_vector[1], edge_vector[2] };

      tensor3 += beta * edge_length * eev * eev.transpose();
    }

    Eigen::SelfAdjointEigenSolver<Matrix3S> solver(tensor3);

    // reorder eigenvalues according to their absolute value.
    std::array<int, 3> order{ 0, 1, 2 };
    std::array<Scalar, 3> abs_lambda;
    abs_lambda[0] = abs(solver.eigenvalues()[0]);
    abs_lambda[1] = abs(solver.eigenvalues()[1]);
    abs_lambda[2] = abs(solver.eigenvalues()[2]);
    if (abs_lambda[0] <= abs_lambda[1] && abs_lambda[0] <= abs_lambda[2]) {
      order[0] = 0;
      if (abs_lambda[1] <= abs_lambda[2]) {
        order[1] = 1;
        order[2] = 2;
      }
      else {
        order[1] = 2;
        order[2] = 1;
      }
    }
    else if (abs_lambda[1] <= abs_lambda[0] && abs_lambda[1] <= abs_lambda[2]) {
      order[0] = 1;
      if (abs_lambda[0] <= abs_lambda[2]) {
        order[1] = 0;
        order[2] = 2;
      }
      else {
        order[1] = 2;
        order[2] = 0;
      }
    }
    else {
      order[0] = 2;
      if (abs_lambda[0] <= abs_lambda[1]) {
        order[1] = 0;
        order[2] = 1;
      }
      else {
        order[1] = 1;
        order[2] = 0;
      }
    }

    Matrix3S V = solver.eigenvectors();
    Normal d1{ V(0, order[2]), V(1, order[2]) , V(2, order[2]) };
    Normal d2{ V(0, order[1]), V(1, order[1]) , V(2, order[1]) };

    mesh.data(vh).d1 = d1;
    mesh.data(vh).d2 = d2;
    mesh.data(vh).k1 = solver.eigenvalues()[order[2]];
    mesh.data(vh).k2 = solver.eigenvalues()[order[1]];
  }
};
```
